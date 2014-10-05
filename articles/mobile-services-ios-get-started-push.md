<properties linkid="develop-mobile-tutorials-get-started-with-push-ios" urlDisplayName="Get Started with Push (iOS)" pageTitle="Get started with push notifications (iOS) | Mobile Dev Center" metaKeywords="" description="Learn how to use Azure Mobile Services to send push notifications to your iOS app." metaCanonical="http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/get-started-with-push-dotnet/" services="" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services" solutions="" manager="dwrede" editor="" authors="ricksal" />

# 移动服务中的推送通知入门

<div class="dev-center-tutorial-selector sublanding">
	<a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-push" title="Windows Store C#">Windows 应用商店 C\#</a>
	<a href="/zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-push" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a>
	<a href="/zh-cn/documentation/articles/mobile-services-windows-phone-get-started-push" title="Windows Phone">Windows Phone</a>
	<a href="/zh-cn/documentation/articles/mobile-services-ios-get-started-push" title="iOS" class="current">iOS</a>
	<a href="/zh-cn/documentation/articles/mobile-services-android-get-started-push" title="Android">Android</a>
	<a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started-push" title="Xamarin.iOS">Xamarin.iOS</a>
	<a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started-push" title="Xamarin.Android">Xamarin.Android</a>
</div>

<div class="dev-onpage-video-clear clearfix">
<div class="dev-onpage-left-content">
<p>本主题说明如何使用 Azure 移动服务向 iOS 应用程序发送推送通知。在本教程中，你将要使用 Apple 推送通知服务 (APNS) 向快速入门项目添加推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。</p>

<p>单击右侧的剪辑可观看本教程的视频版本。</p>

</div>

<div class="dev-onpage-video-wrapper"><a href="http://channel9.msdn.com/posts/iOS-Support-in-Windows-Azure-Mobile-Services" target="_blank" class="label">观看教程</a> <a style="background-image: url('/media/devcenter/mobile/videos/get-started-with-push-ios-180x120.png') !important;" href="http://channel9.msdn.com/posts/iOS-Support-in-Windows-Azure-Mobile-Services" target="_blank" class="dev-onpage-video"><span class="icon">播放视频</span></a> <span class="time">10:37</span></div>
</div>

> [WACOM.NOTE] 本主题说明如何使用移动服务提供的旧版支持来启用推送通知。Azure 通知中心与移动服务相集成，可让你向数百万台设备发送基于模板的跨平台推送通知。默认情况下，使用通知中心的推送通知未启用。

本教程将指导你完成启用推送通知的以下基本步骤：

1.  [生成证书签名请求][]
2.  [注册应用程序和启用推送通知][]
3.  [为应用程序创建配置文件][]
4.  [配置移动服务][]
5.  [向应用程序添加推送通知][]
6.  [更新脚本以发送推送通知][]
7.  [插入数据以接收通知][]

本教程需要的内容如下：

-   [移动服务 iOS SDK][]
-   [XCode 4.5][]
-   支持 iOS 5.0（或更高版本）的设备
-   iOS 开发人员计划成员身份

> [WACOM.NOTE] 根据推送通知配置要求，你必须在支持 iOS 的设备（iPhone 或 iPad）而不是在模拟器上部署和测试推送通知。

本教程基于移动服务快速入门。在开始本教程之前，必须先完成[移动服务入门][]。

Apple 推送通知服务 (APNS) 使用证书来验证你的移动服务。按照以下说明创建必要的证书并将其上载到你的移动服务。有关正式的 APNS 功能文档，请参阅 [Apple 推送通知服务][]。

## 生成证书签名请求文件

首先，你必须生成证书签名请求 (CSR) 文件，Apple 将使用该文件生成签名证书。

1.  从 Utilities 文件夹中，运行 Keychain Access 工具。

2.  单击“Keychain Access” ，展开“Certificate Assistant”（证书助理） ，然后单击“Request a Certificate from a Certificate Authority...”（从证书颁发机构请求证书...） 。

    ![][]

3.  选择你的“User Email Address”（用户电子邮件地址）和“Common Name”（公用名） ，确保已选择“Saved to disk”（保存到磁盘） ，然后单击“Continue”（继续） 。将“CA Email Address”（CA 电子邮件地址）字段保留空白，因为它不是必填字段 。

    ![][1]

4.  在“Save As”（另存为）中为证书签名请求 (CSR) 文件键入一个名称 ，在“Where”（位置） 中选择一个位置，然后单击“Save”（保存） 。

    ![][2]

    此操作会将 CSR 文件保存到选定位置；默认位置是桌面。请记住为此文件选择的位置。

接下来，你将要向 Apple 注册你的应用程序、启用推送通知并上载这个导出的 CSR 以创建一个推送证书。

## 为推送通知注册应用程序

若要将推送通知从移动服务发送到 iOS 应用程序，你必须向 Apple 注册应用程序，还要注册推送通知。

1.  如果你尚未注册应用程序，请导航到 Apple 开发人员中心的 [iOS 设置门户][]，使用 Apple ID 登录，单击“Identifiers”（标识符） ，然后单击“App IDs”（应用程序 ID） ，最后单击“+” 符号以注册新的应用程序。

    ![][3]

2.  在“Description”（说明） 中为应用程序键入一个名称，在“Bundle Identifier”（捆绑标识符） 中输入值 *MobileServices.Quickstart*，在“App Services”（应用程序服务）部分中选中“Push Notifications”（推送通知）选项，然后单击“Continue”（继续） 。此示例将使用 ID "MobileServices.Quickstart"，但你不可以重用这个 ID，因为应用程序 ID 在所有用户之间必须唯一。因此，建议在应用程序名称的后面附加完整名称或首字母。

    ![][4]

    此时将会生成你的应用程序 ID 并请求你"提交"该信息。单击“Submit”（提交） 。

    ![][5]

    单击“Submit”（提交） 后，你将会看到如下所示的“Registration complete”（注册已完成） 屏幕。单击“Done”（完成） 。

    ![][6]

    > [WACOM.NOTE] 如果你选择提供“Bundle Identifier”（捆绑标识符） 值，而不是 *MobileServices.Quickstart*，则还必须更新 Xcode 项目中的捆绑标识符值。

3.  找到你刚刚创建的应用程序 ID，然后单击其行。

    ![][7]

    单击应用程序 ID 将显示有关应用程序和应用程序 ID 的详细信息。单击“Settings”（设置）按钮 。

    ![][8]

4.  滚动到屏幕底部并单击“Development Push SSL Certificate”（开发推送 SSL 证书 ） 部分下的“Create Certificate...”（创建证书...）按钮 。

    ![][9]

    将显示“Add iOS Certificate”（添加 iOS 证书）助手。

    ![][9]

    > [WACOM.NOTE] 本教程使用开发证书。注册生产证书时使用相同的过程。将证书上载至移动服务时，只需确保设置了相同的证书类型即可。

5.  单击“Choose File”（选择文件） ，浏览到你在第一个任务中创建的 CSR 文件保存到的位置，然后单击“Generate”（生成） 。

    ![][10]

6.  门户创建证书之后，请单击“Download”（下载）按钮 ，然后单击“Done”（完成） 。

    ![][11]

    随后将会下载签名证书并将其保存到计算机上的 Downloads 文件夹。

    ![][12]

    > [WACOM.NOTE] 默认情况下，下载的文件（开发证书）名为 "aps\_development.cer"。

7.  双击下载的推送证书 "aps\_development.cer"。

    将在 Keychain 中安装新证书，如下所示：

    ![][13]

    > [WACOM.NOTE] 证书中的名称可能不同，但将以 "Apple Development iOS Push Notification Services:" 作为前缀。

稍后，你将要使用此证书生成一个 .p12 文件，并将其上载到移动服务以使用 APNS 启用身份验证。

## 为应用程序创建配置文件

1.  返回 [iOS 设置门户][]，选择“Provisioning Profiles”（设置配置文件） ，选择“All”（全部） ，然后单击“+” 按钮创建一个新的配置文件。此时会启动“Add iOS Provisiong Profile”（添加 iOS 设置配置文件） 向导

    ![][14]

2.  选择“Development”（开发） 下的“iOS App Development”（iOS 应用程序开发） 作为设置配置文件类型，然后单击“Continue”（继续） 。

    ![][15]

3.  接下来，从“App ID”（应用程序 ID） 下拉列表中选择移动服务快速入门应用程序的应用程序 ID，然后单击“Continue”（继续） 。

    ![][16]

4.  在“Select certificates”（选择证书） 屏幕中，选择前面创建的证书，然后单击“Continue”（继续） 。

    ![][17]

5.  接下来，选择要用于测试的“Devices”（设备） ，然后单击“Continue”（继续） 。

    ![][18]

6.  最后，在“Profile Name”（配置文件名称） 中为配置文件选取一个名称，单击“Generate”（生成） ，然后单击“Done”（完成） 。

    ![][19]

    此操作可创建新的配置文件。

7.  在 Xcode 中，打开“Organizer”（组织程序）并选择“Devices”（设备）视图，在左窗格的“Library”（库） 部分选择“Provisioning Profiles”（配置文件） ，然后单击中间窗格底部的“刷新” 按钮。

    ![][20]

8.  在“Targets”（目标） 下单击“Quickstart”（快速入门） ，展开“Code Signing Identity”（代码签名标识） ，然后在“Debug”（调试） 下选择新的配置文件。

    ![][21]

这可以确保 Xcode 项目使用新配置文件进行代码签名。接下来，必须将该证书上载到移动服务。

## 配置移动服务以发送推送请求

将应用程序注册到 APNS 并配置项目后，接下来必须配置移动服务以便与 APNS 集成。

1.  在 Keychain Access 中，右键单击新证书，单击“Export”（导出） ，为 QuickstartPusher 文件命名，选择 ".p12" 格式，然后单击“Save”（保存） 。

    ![][22]

记下文件名和导出证书的位置。

    > [WACOM.NOTE] 本教程将创建一个 QuickstartPusher.p12 文件。你的文件名和位置可以不同。

1.  登录到 [Azure 管理门户][]，单击“移动服务” ，然后单击你的应用程序。

    ![][23]

2.  单击“推送”选项卡，然后单击“上载” 。

    ![][24]

    此时将显示“上载证书”对话框。

3.  单击“文件”，选择导出的 QuickstartPusher.p12 证书文件，输入密码，确保已选择正确的“模式”，单击勾选图标，然后单击“保存” 。

    ![][25]

    > [WACOM.NOTE] 本教程使用开发证书。

现在，你的移动服务已配置为使用 APNS。

## 向应用程序添加推送通知

1.  在 Xcode 中，打开 QSAppDelegate.h 文件并在 \"*window" 属性下面添加以下属性：

        @property (strong, nonatomic) NSString *deviceToken;

    > [WACOM.NOTE] 如果在移动服务中启用了动态架构，则插入包含此属性的新项时，将在 "TodoItem" 表中自动添加新的“deviceToken”列。

2.  在 QSAppDelegate.m 中，替换实现中的以下处理程序方法：

        - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:
        (NSDictionary *)launchOptions
        {
        // Register for remote notifications
        [[UIApplication sharedApplication] registerForRemoteNotificationTypes:
        UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];
        return YES;
        }

3.  在 QSAppDelegate.m 中，在实现中添加以下处理程序方法：

        // We are registered, so now store the device token (as a string) on the AppDelegate instance
        // taking care to remove the angle brackets first.
        - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:
        (NSData *)deviceToken {
        NSCharacterSet *angleBrackets = [NSCharacterSet characterSetWithCharactersInString:@"<>"];
        self.deviceToken = [[deviceToken description] stringByTrimmingCharactersInSet:angleBrackets];
        }

4.  在 QSAppDelegate.m 中，在实现中添加以下处理程序方法：

        // Handle any failure to register.In this case we set the deviceToken to an empty
        // string to prevent the insert from failing.
        - (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:
        (NSError *)error {
        NSLog(@"Failed to register for remote notifications:%@", error);
        self.deviceToken = @"";
        }

5.  在 QSAppDelegate.m 中，在实现中添加以下处理程序方法：

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

6.  在 QSTodoListViewController.m 中，导入 QSAppDelegate.h 文件，以便能够使用委托来获取设备标记：

        #import "QSAppDelegate.h"

7.  在 QSTodoListViewController.m 中，找到以下行并修改 "(IBAction)onAdd" 操作：

        NSDictionary *item = @{ @"text" :itemText.text, @"complete" :@(NO) }; 

将此代码替换为以下代码：

        // Get a reference to the AppDelegate to easily retrieve the deviceToken
    QSAppDelegate *delegate = [[UIApplication sharedApplication] delegate];

    NSDictionary *item = @{
    @"text" :itemText.text,
    @"complete" :@(NO),
    // add the device token property to our todo item payload
    @"deviceToken" :delegate.deviceToken
        };

    This adds a reference to the "QSAppDelegate" to obtain the device token and then modifies the request payload to include that device token.

    > [WACOM.NOTE] You must add this code before to the call to the <strong>addItem</strong> method.

你的应用程序现已更新，可支持推送通知。

## 在管理门户中更新已注册的插入脚本

1.  在管理门户中，单击“数据”选项卡，然后单击“TodoItem”表 。

    ![][26]

2.  在“todoitem” 中，单击“脚本” 选项卡，然后选择“插入” 。

    ![][27]

    将显示当 "TodoItem" 表中发生插入时所调用的函数。

3.  将 insert 函数替换为以下代码，然后单击“保存”： 

        function insert(item, user, request) {
        request.execute();
        // Set timeout to delay the notification, to provide time for the 
        // app to be closed on the device to demonstrate toast notifications
        setTimeout(function() {
        push.apns.send(item.deviceToken, {
        alert:"Toast:" + item.text,
        payload: {
        inAppMessage:"Hey, a new item arrived:'" + item.text + "'"
                    }
                });
            }, 2500);
        }

    这将会注册一个新的插入脚本，该脚本使用 [apns 对象][]将推送通知（插入的文本）发送到插入请求中提供的设备。

    > [WACOM.NOTE] 此脚本将延迟发送通知，使你有足够的时间关闭应用程序以接收 toast 通知。

## 在应用程序中测试推送通知

1.  在支持 iOS 的设备中按“运行”按钮以生成项目并启动应用程序，然后单击“确定”接受推送通知 

    ![][28]

    > [WACOM.NOTE] 你必须显式接受来自应用程序的推送通知。此请求只会在首次运行应用程序时出现。

2.  在应用程序中键入有意义的文本（例如 *A new Mobile Services task*），然后单击加号 ("+") 图标。

    ![][29]

3.  检查是否已收到通知，然后单击“确定”以取消通知 。

    ![][30]

4.  重复步骤 2 并立即关闭应用程序，然后检查是否已显示以下 toast。

    ![][31]

你已成功完成本教程。

## 后续步骤

在这个简单的示例中，用户将会收到包含刚刚插入的数据的推送通知。请求中的客户端会将 APNS 使用的设备标记提供给移动服务。在下一教程[向应用程序用户推送通知][]中，你将要创建一个单独的 Devices 表，该表用于存储设备标记，以及在发生插入操作时向所有存储的通道发出推送通知。

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-push "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-push "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-push "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/mobile-services-ios-get-started-push "iOS"
  [Android]: /zh-cn/documentation/articles/mobile-services-android-get-started-push "Android"
  [Xamarin.iOS]: /zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started-push "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started-push "Xamarin.Android"
  [观看教程]: http://channel9.msdn.com/posts/iOS-Support-in-Windows-Azure-Mobile-Services
  [生成证书签名请求]: #certificates
  [注册应用程序和启用推送通知]: #register
  [为应用程序创建配置文件]: #profile
  [配置移动服务]: #configure
  [向应用程序添加推送通知]: #add-push
  [更新脚本以发送推送通知]: #update-scripts
  [插入数据以接收通知]: #test
  [移动服务 iOS SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
  [XCode 4.5]: https://go.microsoft.com/fwLink/p/?LinkID=266532
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started-ios
  [Apple 推送通知服务]: http://go.microsoft.com/fwlink/p/?LinkId=272584
  []: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step5.png
  [1]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step6.png
  [2]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step7.png
  [iOS 设置门户]: http://go.microsoft.com/fwlink/p/?LinkId=272456
  [3]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-02.png
  [4]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-03.png
  [5]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-04.png
  [6]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-05.png
  [7]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-06.png
  [8]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-07.png
  [9]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-08.png
  [10]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-10.png
  [11]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-11.png
  [12]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step9.png
  [13]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step10.png
  [14]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-12.png
  [15]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-13.png
  [16]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-14.png
  [17]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-15.png
  [18]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-16.png
  [19]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-17.png
  [20]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-01.png
  [21]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step17.png
  [22]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step18.png
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [23]: ./media/mobile-services-ios-get-started-push/mobile-services-selection.png
  [24]: ./media/mobile-services-ios-get-started-push/mobile-push-tab-ios.png
  [25]: ./media/mobile-services-ios-get-started-push/mobile-push-tab-ios-upload.png
  [26]: ./media/mobile-services-ios-get-started-push/mobile-portal-data-tables.png
  [27]: ./media/mobile-services-ios-get-started-push/mobile-insert-script-push2.png
  [apns 对象]: http://go.microsoft.com/fwlink/p/?LinkId=272333
  [28]: ./media/mobile-services-ios-get-started-push/mobile-quickstart-push1-ios.png
  [29]: ./media/mobile-services-ios-get-started-push/mobile-quickstart-push2-ios.png
  [30]: ./media/mobile-services-ios-get-started-push/mobile-quickstart-push3-ios.png
  [31]: ./media/mobile-services-ios-get-started-push/mobile-quickstart-push4-ios.png
  [向应用程序用户推送通知]: /zh-cn/develop/mobile/tutorials/push-notifications-to-users-ios
