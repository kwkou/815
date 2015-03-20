<properties linkid="develop-mobile-tutorials-get-started-with-push-ios" urlDisplayName="Get Started with Push (iOS)" pageTitle="推送通知入门 (iOS) | 移动开发人员中心" metaKeywords="" description="了解如何使用 Azure 移动服务向 iOS 应用程序发送推送通知。" metaCanonical="http://www.windowsazure.cn/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push/" services="" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services" solutions="" manager="dwrede" editor="" authors="ricksal" />

 
# 将推送通知添加到移动服务应用程序（旧推送）
<div class="dev-center-tutorial-selector sublanding">
    <a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-push" title="Windows Store C#">Windows 应用商店 C#</a>
    <a href="/zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-push" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a>
    <a href="/zh-cn/documentation/articles/mobile-services-windows-phone-get-started-push" title="Windows Phone">Windows Phone</a>
    <a href="/zh-cn/documentation/articles/mobile-services-ios-get-started-push" title="iOS" class="current">iOS</a>
    <a href="/zh-cn/documentation/articles/mobile-services-android-get-started-push" title="Android">Android</a>
	<a href="/zh-cn/documentation/articles/partner-appcelerator-mobile-services-javascript-backend-appcelerator-get-started-push" title="Appcelerator">Appcelerator</a>
</div>

<div class="dev-center-tutorial-subselector"><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-get-started-push/" title=".NET backend">.NET 后端</a> | <a href="/zh-cn/documentation/articles/mobile-services-ios-get-started-push/"  title="JavaScript backend" class="current">JavaScript 后端</a></div>

本主题说明如何使用 Azure 移动服务向 iOS 应用程序发送推送通知。在本教程中，你将要使用 Apple 推送通知服务 (APNS) 向快速入门项目添加推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。


>[WACOM.NOTE]本主题支持<em>尚未升级</em>的<em>现有</em>移动服务使用通知中心集成。创建<em>新</em>移动服务时，会自动启用此集成的功能。有关新移动服务，请参阅[推送通知入门](/zh-cn/documentation/articles/mobile-services-javascript-backend-ios-get-started-push/)。
>
>移动服务现在将与 Azure 通知中心集成，以支持附加的推送通知功能，如模板、多个平台和扩大的规模。<em>你应该升级现有的移动服务以在可能的情况下使用通知中心</em>。升级后，请参阅此版本的[推送通知入门](/zh-cn/documentation/articles/mobile-services-javascript-backend-ios-get-started-push/)。

本教程将指导你完成启用推送通知的以下基本步骤：

1. [生成证书签名请求]
2. [注册应用程序和启用推送通知]
3. [为应用程序创建配置文件]
3. [配置移动服务]
4. [向应用程序添加推送通知]
5. [更新脚本以发送推送通知]
6. [插入数据以接收通知]

本教程需要以下项：

+ [移动服务 iOS SDK]
+ [Xcode 4.5][安装 Xcode]
+ 支持 iOS 5.0（或更高版本）的设备
+ iOS 开发人员计划成员身份

   > [WACOM.NOTE] 由于推送通知配置要求，你必须在支持 iOS 的设备（iPhone 或 iPad），而不是在模拟器上部署和测试推送通知。

本教程基于移动服务快速入门。在开始本教程之前，必须先完成[移动服务入门]。

[WACOM.INCLUDE [Enable Apple Push Notifications](../includes/enable-apple-push-notifications.md)]

## 配置移动服务以发送推送请求

[WACOM.INCLUDE [mobile-services-apns-configure-push](../includes/mobile-services-apns-configure-push.md)]

## 向应用程序添加推送通知

1. 在 Xcode 中，打开 QSAppDelegate.h 文件并在 ***window** 属性下面添加以下属性：

        @property (strong, nonatomic) NSString *deviceToken;

    > [WACOM.NOTE] 如果在移动服务中启用了动态架构，则插入包含此属性的新项时，将在 **TodoItem** 表中自动添加新的"deviceToken"列。

2. 在 QSAppDelegate.m 中，替换实现中的以下处理程序方法： 

        - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:
        (NSDictionary *)launchOptions
        {
            // Register for remote notifications
            [[UIApplication sharedApplication] registerForRemoteNotificationTypes:
            UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];
            return YES;
        }

3. 在 QSAppDelegate.m 中，在实现中添加以下处理程序方法： 

        // We are registered, so now store the device token (as a string) on the AppDelegate instance
        // taking care to remove the angle brackets first.
        - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:
        (NSData *)deviceToken {
            NSCharacterSet *angleBrackets = [NSCharacterSet characterSetWithCharactersInString:@"<>"];
            self.deviceToken = [[deviceToken description] stringByTrimmingCharactersInSet:angleBrackets];
        }

4. 在 QSAppDelegate.m 中，在实现中添加以下处理程序方法： 

        // Handle any failure to register. In this case we set the deviceToken to an empty
        // string to prevent the insert from failing.
        - (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:
        (NSError *)error {
            NSLog(@"Failed to register for remote notifications: %@", error);
            self.deviceToken = @"";
        }

5. 在 QSAppDelegate.m 中，在实现中添加以下处理程序方法：  

        // Because toast alerts don't work when the app is running, the app handles them.
        // This uses the userInfo in the payload to display a UIAlertView.
        - (void)application:(UIApplication *)application didReceiveRemoteNotification:
        (NSDictionary *)userInfo {
            NSLog(@"%@", userInfo);
            UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Notification" message:
            [userInfo objectForKey:@"inAppMessage"] delegate:nil cancelButtonTitle:
            @"OK" otherButtonTitles:nil, nil];
            [alert show];
        }

5. 在 QSTodoListViewController.m 中，导入 QSAppDelegate.h 文件，以便能够使用委托来获取设备标记： 

        #import "QSAppDelegate.h"

6. 在 QSTodoListViewController.m 中，找到以下行并修改 **(IBAction)onAdd** 操作： 

        NSDictionary *item = @{ @"text" : itemText.text, @"complete" : @(NO) }; 
 
   Replace this with the following code:

        // Get a reference to the AppDelegate to easily retrieve the deviceToken
        QSAppDelegate *delegate = [[UIApplication sharedApplication] delegate];
    
        NSDictionary *item = @{
            @"text" : itemText.text,
            @"complete" : @(NO),
            // add the device token property to our todo item payload
            @"deviceToken" : delegate.deviceToken
        };

   	这会添加对 **QSAppDelegate** 的引用以获取设备标记，然后修改请求负载以包括该设备标记。

   	> [WACOM.NOTE] 必须将此代码添加到调用 <strong>addItem</strong> 方法之前。

您的应用现已更新，可支持推送通知。

## 在管理门户中更新已注册的插入脚本

1. 在管理门户中，单击"数据"选项卡，然后单击**TodoItem**表。 

   	![][21]

2. 在 **TodoItem** 中，单击"脚本"选项卡，然后选择"插入"。
   
  	![][22]

   	将显示当 **TodoItem** 表中发生插入时所调用的函数。

3. 将 insert 函数替换为以下代码，然后单击"保存"：

        function insert(item, user, request) {
            request.execute();
            // Set timeout to delay the notification, to provide time for the 
            // app to be closed on the device to demonstrate toast notifications
            setTimeout(function() {
                push.apns.send(item.deviceToken, {
                    alert: "Toast: " + item.text,
                    payload: {
                        inAppMessage: "Hey, a new item arrived: '" + item.text + "'"
                    }
                });
            }, 2500);
        }

   	这将会注册一个新的插入脚本，该脚本使用 [apns 对象]将推送通知（插入的文本）发送到插入请求中提供的设备。 


   	> [WACOM.NOTE] 此脚本将延迟发送通知，使你有足够的时间关闭应用程序以接收 toast 通知。

## 在应用程序中测试推送通知

1. 在支持 iOS 的设备中按"运行"按钮以生成项目并启动应用程序，然后单击"确定"以接受推送通知

  	![][23]

    > [WACOM.NOTE] 你必须显式接受来自应用程序的推送通知。此请求只会在首次运行应用程序时出现。

2. 在应用程序中键入有意义的文本（例如 _A new Mobile Services task_），然后单击加号 (**+**) 图标。

  	![][24]

3. 检查是否已收到通知，然后单击"确定"以取消通知。

  	![][25]

4. 重复步骤 2 并立即关闭应用程序，然后检查是否已显示以下 toast。

  	![][26]

你已成功完成本教程。

## 后续步骤

在这个简单的示例中，用户将会收到包含刚刚插入的数据的推送通知。请求中的客户端会将 APNS 使用的设备标记提供给移动服务。在下一教程[向应用程序用户推送通知]中，你将要创建一个单独的 Devices 表，该表用于存储设备标记，以及在发生插入操作时向所有存储的通道发出推送通知。 

<!-- Anchors. -->
[生成证书签名请求]: #certificates
[注册应用程序和启用推送通知]: #register
[为应用程序创建配置文件]: #profile
[配置移动服务]: #configure
[更新脚本以发送推送通知]: #update-scripts
[向应用程序添加推送通知]: #add-push
[插入数据以接收通知]: #test
[后续步骤]:#next-steps

<!-- Images. -->
[5]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step5.png
[6]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step6.png
[7]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step7.png

[9]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step9.png
[10]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step10.png
[17]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step17.png
[18]: ./media/mobile-services-ios-get-started-push/mobile-services-selection.png
[19]: ./media/mobile-services-ios-get-started-push/mobile-push-tab-ios.png
[20]: ./media/mobile-services-ios-get-started-push/mobile-push-tab-ios-upload.png
[21]: ./media/mobile-services-ios-get-started-push/mobile-portal-data-tables.png
[22]: ./media/mobile-services-ios-get-started-push/mobile-insert-script-push2.png
[23]: ./media/mobile-services-ios-get-started-push/mobile-quickstart-push1-ios.png
[24]: ./media/mobile-services-ios-get-started-push/mobile-quickstart-push2-ios.png
[25]: ./media/mobile-services-ios-get-started-push/mobile-quickstart-push3-ios.png
[26]: ./media/mobile-services-ios-get-started-push/mobile-quickstart-push4-ios.png
[28]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step18.png

[101]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-01.png
[102]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-02.png
[103]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-03.png
[104]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-04.png
[105]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-05.png
[106]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-06.png
[107]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-07.png
[108]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-08.png

[110]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-10.png
[111]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-11.png
[112]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-12.png
[113]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-13.png
[114]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-14.png
[115]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-15.png
[116]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-16.png
[117]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-17.png

<!-- URLs. -->
[安装 Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[iOS 设置门户]: http://go.microsoft.com/fwlink/p/?LinkId=272456
[移动服务 iOS SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[Apple 推送通知服务]: http://go.microsoft.com/fwlink/p/?LinkId=272584
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-ios
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-data-ios
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-users-ios
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-ios
[向应用程序用户推送通知]: /zh-cn/documentation/articles/mobile-services-ios-push-notifications-app-users
[使用脚本为用户授权]: /zh-cn/documentation/articles/mobile-services-ios-authorize-users-in-scripts
[Azure 管理门户]: https://manage.windowsazure.cn/
[apns 对象]: http://go.microsoft.com/fwlink/p/?LinkId=272333

