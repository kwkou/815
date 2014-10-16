<properties linkid="develop-mobile-tutorials-build-realtime-apps-with-pusher-ios" urlDisplayName="Build Realtime Apps with Pusher" pageTitle="Build Realtime Apps with Pusher (iOS) - Mobile Services" metaKeywords="" description="Learn how to use Pusher to send notifications to your Azure Media Services app on iOS." metaCanonical="" services="" documentationCenter="Mobile" title="Build Real-time Apps with Mobile Services and Pusher" authors="" solutions="" manager="" editor="" />

# 使用移动服务和 Pusher 生成实时应用程序

<div class="dev-center-tutorial-selector sublanding"> 
	<a href="" title="iOS" class="current">iOS</a> 
</div>

本主题说明如何将实时功能添加到基于 Azure 移动服务的应用程序。完成本主题后，你的 TodoList 数据将在所有运行的应用程序实例之间实时同步。

[向用户推送通知][]教程说明了如何使用推送通知来让用户知道 Todo 列表中已添加了新项。推送通知是显示偶发性更改的极佳方式。但是，应用程序有时需要频繁发送实时通知。在此情况下，你可以使用 Pusher API 将实时通知添加到你的移动服务。在本教程中，我们将结合使用 Pusher 和移动服务，使 Todo 列表在运行的应用程序实例发生更改时保持同步。

Pusher 是一个基于云的服务，与移动服务一样，它可以让你无比轻松地生成实时应用程序。你可以使用 Pusher 快速生成实时投票、聊天室、多玩家游戏和协作应用程序，广播实时数据和内容，但这只是它的一些最基本功能！有关详细信息，请参阅 [][]<http://pusher.com>。

本教程将指导你完成在 Todo 列表应用程序中添加实时协作功能的以下基本步骤：

1.  [创建 Pusher 帐户][]
2.  [更新应用程序][]
3.  [安装服务器脚本][]
4.  [测试应用程序][]

本教程基于移动服务快速入门。在开始本教程之前，必须先完成[移动服务入门][]。

<a name="sign-up"></a>
## 创建新的 Pusher 帐户

[WACOM.INCLUDE [pusher-sign-up](../includes/pusher-sign-up.md)]

<a name="update-app"></a>
## 更新应用程序

设置 Pusher 帐户后，下一步就是修改 iOS 应用程序代码以使用新功能。

### 安装 libPusher 库

[libPusher][] 库可让你从 iOS 访问 Pusher。

1.  从[此处][]下载 libPusher 库。

2.  在项目中创建一个名为 *libPusher* 的组。

3.  在查找工具中，解压缩下载的 zip 文件，选择 "libPusher-combined.a" 和 "/headers" 文件夹，然后将这些项拖放到项目中的 "libPusher" 组内。

4.  选中“将项复制到目标组的文件夹中” ，然后单击“完成” 。

    ![][0]

这就会将 libPusher 文件复制到你的项目中。

1.  在项目资源管理器中的项目根目录位置，单击“生成阶段”，然后单击“添加生成阶段”和“添加复制文件” 。

2.  将 "libPusher-combined.a" 文件从项目资源管理器拖放到新的生成阶段。

3.  将“目标”更改为“框架”，然后单击“仅在安装时复制” 。

    ![][1]

4.  在“将二进制文件链接到库”区域中添加以下库 ：

    -   libicucore.dylib
    -   CFNetwork.framework
    -   Security.framework
    -   SystemConfiguration.framework

5.  最后，在“生成设置”中，找到目标生成设置“其他链接器标志”，并添加 "-all\_load" 标志 。

    ![][2]

    此时将显示针对“调试”生成目标设置的 "-all\_load" 标志。

该库现已安装并可供使用。

### 向应用程序添加代码

1.  在 Xcode 中，打开 "QSTodoService.h" 文件并添加以下方法声明：

        // Allows retrieval of items by id
        - (NSUInteger) getItemIndex:(NSDictionary *)item;

        // To be called when items are added by other users
        - (NSUInteger) itemAdded:(NSDictionary *)item;

        // To be called when items are completed by other users
        - (NSUInteger) itemCompleted:(NSDictionary *)item;

2.  将 "addItem" 和 "completeItem" 的现有声明替换为以下代码：

        - (void) addItem:(NSDictionary *) item;
        - (void) completeItem:(NSDictionary *) item;

3.  在 "QSTodoService.m" 中，添加以下代码以实现新方法：

        // Allows retrieval of items by id
        - (NSUInteger) getItemIndex:(NSDictionary *)item
        {
        NSInteger itemId = [[item objectForKey:@"id"] integerValue];

        return [items indexOfObjectPassingTest:^BOOL(id currItem, NSUInteger idx, BOOL *stop)
                 {
        return ([[currItem objectForKey:@"id"] integerValue] == itemId);
                 }];
        }

        // To be called when items are added by other users
        -(NSUInteger) itemAdded:(NSDictionary *)item
        {
        NSUInteger index = [self getItemIndex:item];

        // Only complete action if item not already in list
        if(index == NSNotFound)
            {
        NSUInteger newIndex = [items count];
        [(NSMutableArray *)items insertObject:item atIndex:newIndex];
        return newIndex;
            }
        else
        return -1;
        }

        // To be called when items are completed by other users
        - (NSUInteger) itemCompleted:(NSDictionary *)item
        {
        NSUInteger index = [self getItemIndex:item];

        // Only complete action if item exists in items list
        if(index != NSNotFound)
            {
        NSMutableArray *mutableItems = (NSMutableArray *) items;
        [mutableItems removeObjectAtIndex:index];
            }       
        return index;
        }

    现在，QSTodoService 允许你按 "id" 查找项以及在本地添加和完成项，而无需向远程服务发送显式请求。

4.  将现有的 "addItem" 和 "completeItem" 方法替换为以下代码：

        -(void) addItem:(NSDictionary *)item
        {
        // Insert the item into the TodoItem table and add to the items array on completion
        [self.table insert:item completion:^(NSDictionary *result, NSError *error) {
        [self logErrorIfNotNil:error];
            }];
        }

        -(void) completeItem:(NSDictionary *)item
        {
        // Set the item to be complete (we need a mutable copy)
        NSMutableDictionary *mutable = [item mutableCopy];
        [mutable setObject:@(YES) forKey:@"complete"];

        // Update the item in the TodoItem table and remove from the items array on completion
        [self.table update:mutable completion:^(NSDictionary *item, NSError *error) {
        [self logErrorIfNotNil:error];
            }];
        }

    请注意，项现已添加并已完成，同时，在收到来自 Pusher 的事件（而不是更新表数据）时，UI 将会更新。

5.  在 "QSTodoListViewController.h" 文件中添加以下 import 语句：

        #import "PTPusherDelegate.h"
        #import "PTPusher.h"
        #import "PTPusherEvent.h"
        #import "PTPusherChannel.h"

6.  修改接口声明以添加类似于下面的 "PTPusherDelegate"：

        @interface QSTodoListViewController :UITableViewController<UITextFieldDelegate, PTPusherDelegate>

7.  添加以下新属性：

        @property (nonatomic, strong) PTPusher *pusher;

8.  添加以下用于声明新方法的代码：

        // Sets up the Pusher client
        - (void) setupPusher;

9.  在 "QSTodoListViewController.m" 中，在其他

    \*\* at synthesise\*

    \* 行下面添加以下行，以实现新属性：

        @synthesize pusher = _pusher;

10. 现在，请添加以下代码以实现新方法：

        // Sets up the Pusher client
        - (void) setupPusher {

        // Create a Pusher client, using your Pusher app key as the credential
        // TODO:Move Pusher app key to configuration file
        self.pusher = [PTPusher pusherWithKey:@""your_app_key"" delegate:self encrypted:NO];
        self.pusher.reconnectAutomatically = YES;

        // Subscribe to the 'todo-updates' channel
        PTPusherChannel *todoChannel = [self.pusher subscribeToChannelNamed:@"todo-updates"];

        // Bind to the 'todo-added' event
        [todoChannel bindToEventNamed:@"todo-added" handleWithBlock:^(PTPusherEvent *channelEvent) {

        // Add item to the todo list
        NSUInteger index = [self.todoService itemAdded:channelEvent.data];

        // If the item was not already in the list, add the item to the UI
        if(index != -1)
                {
        NSIndexPath *indexPath = [NSIndexPath indexPathForRow:index inSection:0];
        [self.tableView insertRowsAtIndexPaths:@[ indexPath ]
        withRowAnimation:UITableViewRowAnimationTop];
                }
            }];

        // Bind to the 'todo-completed' event
        [todoChannel bindToEventNamed:@"todo-completed" handleWithBlock:^(PTPusherEvent *channelEvent) {

        // Update the item to be completed
        NSUInteger index = [self.todoService itemCompleted:channelEvent.data];

        // As long as the item did exit in the list, update the UI
        if(index != NSNotFound)
                {
        NSIndexPath *indexPath = [NSIndexPath indexPathForRow:index inSection:0];
        [self.tableView deleteRowsAtIndexPaths:@[ indexPath ]
        withRowAnimation:UITableViewRowAnimationTop];
                }               
            }];
        }

11. 将 `"your_app_key"` 占位符替换为你前面从“连接信息”对话框中复制的 app\_key 值。

12. 将 "onAdd" 方法替换为以下代码：

        - (IBAction)onAdd:(id)sender
        {
        if (itemText.text.length  == 0) {
        return;
            }

        NSDictionary *item = @{ @"text" :itemText.text, @"complete" :@(NO) };
        [self.todoService addItem:item];

        itemText.text = @"";
        }

13. 在 "QSTodoListViewController.m" 文件中，找到 (void)viewDidLoad 方法并添加对 "setupPusher" 方法的调用，使最前面的新行是：

        - (void)viewDidLoad
        {
        [super viewDidLoad];
        [self setupPusher];

14. 在 "tableView:commitEditingStyle:forRowAtIndexPath" 方法的末尾，将对 "completeItem" 的调用替换为以下代码：

        // Ask the todoService to set the item's complete value to YES
        [self.todoService completeItem:item];

现在，该应用程序可以从 Pusher 接收事件，并可相应地更新本地 Todo 列表。

<a name="install-scripts"></a>
## 安装服务器脚本

剩下的最后一个任务就是设置你的服务器脚本。我们将要插入一个脚本，该脚本将在 TodoList 表中插入或更新项时执行。

1.  登录到 [Azure 管理门户][]，单击“移动服务”，然后单击你的移动服务 。

2.  在管理门户中，单击“数据”选项卡，然后单击“TodoItem”表 。

    ![][3]

3.  在“TodoItem” 中，单击“脚本” 选项卡，然后选择“插入” 。

    ![][4]

    将显示当 "TodoItem" 表中发生插入时所调用的函数。

4.  将插入函数替换为以下代码：

        var Pusher = require('pusher');

        function insert(item, user, request) {   

        request.execute({
        success:function() {
        // After the record has been inserted, trigger immediately to the client
        request.respond();

        // Publish event for all other active clients
        publishItemCreatedEvent(item);
                }
            });

        function publishItemCreatedEvent(item) {

        // Ideally these settings would be taken from config
        var pusher = new Pusher({
        appId:'"your_app_id"',
        key:'"your_app_key"',
        secret:'"your_app_secret"'
                });     

        // Publish event on Pusher channel
        pusher.trigger( 'todo-updates', 'todo-added', item );   
            }
        }

5.  将上述脚本中的占位符替换为你前面从“连接信息”对话框中复制的值。

    -   "`"your_app_id"`"：app\_id 值
    -   "`"your_app_key"`"：app\_key 值
    -   "`"your_app_key_secret"`"：app\_key\_secret

6.  单击“保存” 按钮。现在你已配置了一个脚本，每当在 "TodoItem" 表中插入一个新项时，该脚本都会向 Pusher 发布事件。

7.  从“操作”下拉列表中选择“更新” 。

8.  将 update 函数替换为以下代码：

        var Pusher = require('pusher');

        function update(item, user, request) {   

        request.execute({
        success:function() {
        // After the record has been updated, trigger immediately to the client
        request.respond();

        // Publish event for all other active clients
        publishItemUpdatedEvent(item);
                }
            });

        function publishItemUpdatedEvent(item) {

        // Ideally these settings would be taken from config
        var pusher = new Pusher({
        appId:'"your_app_id"',
        key:'"your_app_key"',
        secret:'"your_app_secret"'
                });     

        // Publish event on Pusher channel
        pusher.trigger( 'todo-updates', 'todo-completed', item );

            }
        }

9.  针对此脚本重复步骤 5 以替换占位符。

10. 单击“保存” 按钮。现在你已配置了一个脚本，每当更新某个新项时，该脚本都会向 Pusher 发布事件。

<a name="test-app"></a>
## 测试应用程序

若要测试应用程序，你需要运行两个实例。可以在 iOS 设备上运行一个实例，在 iOS 模拟器上运行另一个实例。

1.  连接你的 iOS 设备，按“运行”按钮（或者按 Command+R 键）在设备上启动该应用程序，然后停止调试 。

    你的应用程序现已安装到设备上。

2.  在 iOS 模拟器上运行该应用程序，同时在 iOS 设备上启动该应用程序。

    现在，你已运行了该应用程序的两个实例。

3.  在其中一个应用程序实例中添加一个新的 Todo 项。

    检查添加的项是否出现在另一个实例中。

4.  在一个应用程序实例中选中某个 Todo 项可将它标记为已完成。

    检查该项是否已从另一个实例中消失。

祝贺你！你已成功配置了你的移动服务应用程序，现在它可以在所有客户端之间实时同步。

<a name="nextsteps"> </a>
## 后续步骤

现在你已知道，将 Pusher 服务与移动服务结合使用是多么的容易。请通过以下链接了解有关 Pusher 的详细信息。

-   Pusher API 文档：<http://pusher.com/docs>
-   Pusher 教程：<http://pusher.com/tutorials>

若要了解有关注册和使用服务器脚本的详细信息，请参阅[移动服务服务器脚本参考][]。

  [iOS]:  "iOS"
  [向用户推送通知]: /zh-cn/develop/mobile/tutorials/push-notifications-to-users-ios
  []: http://pusher.com
  [创建 Pusher 帐户]: #sign-up
  [更新应用程序]: #update-app
  [安装服务器脚本]: #install-scripts
  [测试应用程序]: #test-app
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started
  [pusher-sign-up]: ../includes/pusher-sign-up.md
  [libPusher]: http://go.microsoft.com/fwlink/p?LinkId=276999
  [此处]: http://go.microsoft.com/fwlink/p/?LinkId=276998
  [0]: ./media/mobile-services-ios-build-realtime-apps-pusher/pusher-ios-add-files-to-group.png
  [1]: ./media/mobile-services-ios-build-realtime-apps-pusher/pusher-ios-add-build-phase.png
  [2]: ./media/mobile-services-ios-build-realtime-apps-pusher/pusher-ios-add-linker-flag.png
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [3]: ./media/mobile-services-ios-build-realtime-apps-pusher/mobile-portal-data-tables.png
  [4]: ./media/mobile-services-ios-build-realtime-apps-pusher/mobile-insert-script-push2.png
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/p/?LinkId=262293
