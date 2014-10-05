<properties linkid="develop-net-tutorials-push-notifications-to-users-ios" urlDisplayName="Push Notifications to Users (iOS)" pageTitle="Push notifications to users (iOS) | Mobile Dev Center" metaKeywords="" description="Learn how to use Mobile Services to push notifications to users of your iOS app." metaCanonical="" services="" documentationCenter="Mobile" title="Push notifications to users by using Mobile Services" authors="" solutions="" manager="" editor="" />

# 使用移动服务向用户推送通知

<div class="dev-center-tutorial-selector sublanding"> 
	<a href="/zh-cn/develop/mobile/tutorials/push-notifications-to-users-wp8" title="Windows Phone">Windows Phone</a><a href="/zh-cn/develop/mobile/tutorials/push-notifications-to-users-ios" title="iOS" class="current">iOS</a><a href="/zh-cn/develop/mobile/tutorials/push-notifications-to-users-android" title="Android">Android</a>
</div>

本主题是[前面的推送通知教程][]的引伸，其中添加了一个用于存储 Apple 推送通知服务 (APNS) 标记的新表。然后，可以使用这些标记向 iPhone 或 iPad 应用程序的用户发送推送通知。

本教程将指导你完成在应用程序中更新推送通知的以下步骤：

1.  [创建 Devices 表][]
2.  [更新应用程序][]
3.  [更新服务器脚本][]
4.  [验证推送通知行为][]

本教程是在移动服务快速入门和前面的[推送通知入门][前面的推送通知教程]教程的基础上制作的。在开始本教程之前，必须先完成[推送通知入门][前面的推送通知教程]。

## 
<a name="create-table"></a>
## 创建表创建新的 Devices 表

1.  登录到 [Azure 管理门户][]，单击“移动服务” ，然后单击你的应用程序。

    ![][]

2.  单击“数据”选项卡 ，然后单击“创建” 。

    ![][1]

    此时将显示“创建新表” 对话框。

3.  为所有权限保留默认的“具有应用程序密钥的任何人”设置，在“表名”中键入 *Devices*，然后单击勾选按钮 。

    ![][2]

    此时将会创建用于存储设备标记的 "Devices" 表，你可以使用这些设备标记发送不同于项数据的推送通知。

接下来，你需要修改推送通知应用程序，以便将数据存储在此新表而不是 "TodoItem" 表中。

<a name="update-app"></a>
## 更新应用程序

1.  在 Xcode 中，打开 QSTodoService.h 文件并添加以下方法声明：

        // Declare method to register device token for other users
        - (void)registerDeviceToken:(NSString *)deviceToken;

    这样，应用程序委托便可以将 deviceToken 注册到移动服务。

2.  在 QSTodoService.m 中，添加以下 instance 方法：

        // Instance method to register deviceToken in Devices table.
        // Called in AppDelegate.m when APNS registration succeeds.
        - (void)registerDeviceToken:(NSString *)deviceToken
        {
        MSTable *devicesTable = [self.client tableWithName:@"Devices"]; 
        NSDictionary *device = @{ @"deviceToken" :deviceToken };

        // Insert the item into the devices table and add to the items array on completion
        [devicesTable insert:device completion:^(NSDictionary *result, NSError *error) {
        if (error) {
        NSLog(@"ERROR %@", error);
                }
            }];
        }

    这样，其他调用方便可以将设备标记注册到移动服务。

3.  在 QSAppDelegate.m 文件中，添加以下 import 语句：

        #import "QSTodoService.h"

    此代码使 AppDelegate 可识别 TodoService 实现。

4.  在 QSAppDelegate.m 中，将 "didRegisterForRemoteNotificationsWithDeviceToken" 方法替换为以下代码：

        // We have registered, so now store the device token (as a string) on the AppDelegate instance
        // taking care to remove the angle brackets first.
        - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:
        (NSData *)deviceToken {

        // Register the APNS deviceToken with the Mobile Service Devices table.
        NSCharacterSet *angleBrackets = [NSCharacterSet characterSetWithCharactersInString:@"<>"];
        NSString *token = [[deviceToken description] stringByTrimmingCharactersInSet:angleBrackets];

        QSTodoService *instance = [QSTodoService defaultService];
        [instance registerDeviceToken:token];
        }

5.  在 QSTodoListViewController.m 中，找到 "(IBAction)onAdd" 方法并*删除*以下代码：

        // Get a reference to the AppDelegate to easily retrieve the deviceToken
        QSAppDelegate *delegate = [[UIApplication sharedApplication] delegate];

        NSDictionary *item = @{
        @"text" :itemText.text,
        @"complete" :@(NO),
        // add the device token property to our todo item payload
        @"deviceToken" :delegate.deviceToken
        };

    将此代码替换为以下代码：

        // We removed the delegate; this application no longer passes the deviceToken here.
        // Remove the device token from the payload
        NSDictionary *item = @{ @"text" :itemText.text, @"complete" :@(NO) };

现在，你的应用程序已更新，可以使用新的 Devices 表来存储用于向设备发回推送通知的设备标记。

<a name="update-scripts"></a>
## 更新服务器脚本

1.  在管理门户中，单击“数据”选项卡，然后单击“Devices”表 。

    ![][3]

2.  在“devices” 中，单击“脚本” 选项卡，然后选择“插入” 。

    ![][4]

    将显示当 "Devices" 表中发生插入时所调用的函数。

3.  将 insert 函数替换为以下代码，然后单击“保存”： 

        function insert(item, user, request) {
        var devicesTable = tables.getTable('Devices');
        devicesTable.where({
        token:item.token
        }).read({
        success:insertTokenIfNotFound
           });

        function insertTokenIfNotFound(existingTokens) {
        if (existingTokens.length > 0) {
        request.respond(200, existingTokens[0]);
        } else {
        request.execute();
               }
           }
        }

    此脚本将检查 "Devices" 表中是否存在具有相同标记的现有设备。仅当未找到匹配的设备时，插入操作才会继续。这可以防止出现重复的设备记录。

4.  单击“TodoItem” ，再单击“脚本”并选择“插入”。 

    ![][5]

5.  将 insert 函数替换为以下代码，然后单击“保存”： 

        function insert(item, user, request) {
        request.execute({
        success:function() {
        request.respond();
        sendNotifications();
              }
          });

        function sendNotifications() {
        var devicesTable = tables.getTable('Devices');
        devicesTable.read({
        success:function(devices) {
        // Set timeout to delay the notifications, 
        // to provide time for the app to be closed 
        // on the device to demonstrate toast notifications.
        setTimeout(function() {
        devices.forEach(function(device) {

        push.apns.send(device.deviceToken, {
        alert:"Toast:" + item.text,
        payload: {
        inAppMessage: 
        "Hey, a new item arrived: '" + 
        item.text + "'"
                                  }
                              });
                          });
                      }, 2500);
                  }
              });
          }

    }

    此插入脚本将向 "Devices" 表中存储的所有设备发送推送通知（包括已插入项的文本）。

<a name="test"></a>
## 测试应用程序在应用程序中测试推送通知

1.  在支持 iOS 的设备中按“运行” 按钮以生成项目并启动应用程序，在应用程序中键入有意义的文本（例如 *A new Mobile Services task*），然后单击加号 ("+") 图标。

    ![][6]

2.  检查是否已收到通知，然后单击“确定”以取消通知 。

    ![][7]

3.  重复步骤 2 并立即关闭应用程序，然后检查是否已显示以下 toast。

    ![][8]

你已成功完成本教程。

## 后续步骤

演示推送通知操作基础知识的教程到此结束。建议你了解有关以下移动服务主题的详细信息：

-   [数据处理入门][]
    了解有关使用移动服务存储和查询数据的详细信息。

-   [身份验证入门][]
    了解如何使用 Windows 帐户对应用程序用户进行身份验证。

-   [移动服务服务器脚本参考][]
    了解有关注册和使用服务器脚本的详细信息。

  [Windows Phone]: /zh-cn/develop/mobile/tutorials/push-notifications-to-users-wp8 "Windows Phone"
  [iOS]: /zh-cn/develop/mobile/tutorials/push-notifications-to-users-ios "iOS"
  [Android]: /zh-cn/develop/mobile/tutorials/push-notifications-to-users-android "Android"
  [前面的推送通知教程]: /zh-cn/develop/mobile/tutorials/get-started-with-push-ios
  [创建 Devices 表]: #create-table
  [更新应用程序]: #update-app
  [更新服务器脚本]: #update-scripts
  [验证推送通知行为]: #test-app
  [Azure 管理门户]: https://manage.windowsazure.cn/
  []: ./media/mobile-services-ios-push-notifications-app-users/mobile-services-selection.png
  [1]: ./media/mobile-services-ios-push-notifications-app-users/mobile-create-table.png
  [2]: ./media/mobile-services-ios-push-notifications-app-users/mobile-create-devices-table.png
  [3]: ./media/mobile-services-ios-push-notifications-app-users/mobile-portal-data-tables-devices.png
  [4]: ./media/mobile-services-ios-push-notifications-app-users/mobile-insert-script-devices.png
  [5]: ./media/mobile-services-ios-push-notifications-app-users/mobile-insert-script-push2.png
  [6]: ./media/mobile-services-ios-push-notifications-app-users/mobile-quickstart-push2-ios.png
  [7]: ./media/mobile-services-ios-push-notifications-app-users/mobile-quickstart-push3-ios.png
  [8]: ./media/mobile-services-ios-push-notifications-app-users/mobile-quickstart-push4-ios.png
  [数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-ios
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-ios
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/?LinkId=262293
