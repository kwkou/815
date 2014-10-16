<properties linkid="develop-notificationhubs-tutorials-get-started-ios" urlDisplayName="Get Started" pageTitle="Get Started with Azure Notification Hubs" metaKeywords="" description="Learn how to use Azure Notification Hubs to push notifications." metaCanonical="" services="notification-hubs" documentationCenter="Mobile" title="Get started with Notification Hubs" authors="sethm" solutions="" manager="" editor="" />

# 通知中心入门

<div class="dev-center-tutorial-selector sublanding"><a href="/en-us/documentation/articles/notification-hubs-windows-store-dotnet-get-started/" title="Windows 应用商店 C#">Windows 应用商店 C#</a><a href="/en-us/documentation/articles/notification-hubs-windows-phone-get-started/" title="Windows Phone">Windows Phone</a><a href="/en-us/documentation/articles/notification-hubs-ios-get-started/" title="iOS" class="current">iOS</a><a href="/en-us/documentation/articles/notification-hubs-android-get-started/" title="Android" class="current">Android</a><a href="/en-us/documentation/articles/notification-hubs-kindle-get-started/" title="Kindle" class="current">Kindle</a><a href="/en-us/documentation/articles/partner-xamarin-notification-hubs-ios-get-started/" title="Xamarin.iOS" class="current">Xamarin.iOS</a><a href="/en-us/documentation/articles/partner-xamarin-notification-hubs-android-get-started/" title="Xamarin.Android" class="current">Xamarin.Android</a></div>

本主题说明如何使用 Azure 通知中心向 iOS 应用程序发送推送通知。
在本教程中，你将要创建一个使用 Apple 推送通知服务 (APNS) 接收推送通知的空白 iOS 应用程序。完成后，你将能使用通知中心将推送通知广播到运行你的应用程序的所有设备。

本教程将指导你完成启用推送通知的以下基本步骤：

1.  [生成证书签名请求][生成证书签名请求]
2.  [注册应用程序和启用推送通知][注册应用程序和启用推送通知]
3.  [为应用程序创建配置文件][为应用程序创建配置文件]
4.  [配置通知中心][配置通知中心]
5.  [将你的应用程序连接到通知中心][将你的应用程序连接到通知中心]
6.  [从后端发送通知][从后端发送通知]

本教程演示使用通知中心的简单广播方案。请确保随后学习下一教程以了解如何使用通知中心来发送到特定用户和设备组。本教程需要满足以下前提条件：

-   [移动服务 iOS SDK][移动服务 iOS SDK]
-   [XCode 4.5][XCode 4.5]
-   支持 iOS 5.0（或更高版本）的设备
-   iOS 开发人员计划成员身份

<div class="dev-callout"><b>说明</b><br /> <p>由于推送通知配置要求，你必须在支持 iOS 的设备（iPhone 或 iPad），而不是在模拟器上部署和测试推送通知。</p><br /> </div>
</p> 完成本教程是学习有关 iOS 应用程序的所有其他通知中心教程的先决条件。
<div class="dev-callout"><strong>说明</strong> <p>若要完成本教程，你必须有一个有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.cn/zh-cn/pricing/free-trial/" target="_blank">Azure 免费试用</a>。</p></div>

Apple 推送通知服务 (APNS) 使用证书来验证你的移动服务。按照以下说明创建必要的证书并将其上载到你的移动服务。有关正式的 APNS 功能文档，请参阅 [Apple 推送通知服务]。

## <a name="certificates"></a><span class="short-header">生成 CSR 文件</span>生成证书签名请求文件

首先，你必须生成证书签名请求 (CSR) 文件，Apple 将使用该文件生成签名证书。

1.  从 Utilities 文件夹中，运行 Keychain Access 工具。

2.  单击“Keychain Access”，展开“Certificate Assistant”（证书助理），然后单击“Request a Certificate from a Certificate Authority...”（从证书颁发机构请求证书...）。

    ![][]

3.  选择你的“User Email Address”（用户电子邮件地址）和“Common Name”（公用名），确保已选择“Saved to disk”（保存到磁盘），然后单击“Continue”（继续）。将“CA Email Address”（CA 电子邮件地址）字段保留空白，因为它不是必填字段。

    ![][1]

4.  在“Save As”（另存为）中为证书签名请求 (CSR) 文件键入一个名称，在“Where”（位置）中选择一个位置，然后单击“Save”（保存）。

    ![][2]

此操作会将 CSR 文件保存到选定位置；默认位置是桌面。请记住为此文件选择的位置。

接下来，你将向 Apple 注册你的应用程序、启用推送通知并上载此导出的 CSR 以创建一个推送证书。

## <a name="register"></a><span class="short-header">注册应用程序</span>为推送通知注册应用程序

若要将推送通知从移动服务发送到 iOS 应用程序，你必须向 Apple 注册应用程序，还要注册推送通知。

1.  如果你尚未注册应用程序，请导航到 Apple 开发人员中心的 [iOS 设置门户][iOS 设置门户]，使用 Apple ID 登录，单击“Identifiers”（标识符），然后单击“App IDs”（应用程序 ID），最后单击“+”符号以注册新的应用程序。

    ![][3]

2.  在“Description”（说明）中为应用程序键入一个名称，在“Bundle Identifier”（捆绑标识符）中输入值 *MobileServices.Quickstart*，在“App Services”（应用程序服务）部分中选中“Push Notifications”（推送通知）选项，然后单击“Continue”（继续）。此示例将使用 ID **MobileServices.Quickstart**，但你不可以重用这个 ID，因为应用程序 ID 在所有用户之间必须唯一。因此，建议在应用程序名称的后面附加完整名称或首字母。

    ![][4]

    此时将会生成你的应用程序 ID 并请求你**提交**该信息。单击“Submit”（提交）。

    ![][5]

    单击“Submit”（提交）后，你将会看到如下所示的“Registration complete”（注册已完成）屏幕。单击“Done”（完成）。

    ![][6]

    > [WACOM.NOTE] 如果你选择提供“Bundle Identifier”（捆绑标识符）值，而不是 *MobileServices.Quickstart*，则还必须更新 Xcode 项目中的捆绑标识符值。

3.  找到你刚刚创建的应用程序 ID，然后单击其行。

    ![][7]

    单击应用程序 ID 将显示有关应用程序和应用程序 ID 的详细信息。单击“Settings”（设置）按钮。

    ![][8]

4.  滚动到屏幕底部并单击“Development Push SSL Certificate”（开发推送 SSL 证书 ）部分下的“Create Certificate...”（创建证书...）按钮。

    ![][9]

    将显示“Add iOS Certificate”（添加 iOS 证书）助手。

    ![][9]

    > [WACOM.NOTE] 本教程使用开发证书。注册生产证书时使用相同的过程。将证书上载至移动服务时，只需确保设置了相同的证书类型即可。

5.  单击“Choose File”（选择文件），浏览到你在第一个任务中创建的 CSR 文件保存到的位置，然后单击“Generate”（生成）。

    ![][10]

6.  门户创建证书之后，请单击“Download”（下载）按钮，然后单击“Done”（完成）。

    ![][11]

    随后将会下载签名证书并将其保存到计算机上的 Downloads 文件夹。

    ![][12]

    > [WACOM.NOTE] 默认情况下，下载的文件（开发证书）名为 **aps\_development.cer**。

7.  双击下载的推送证书 **aps\_development.cer**。

    将在 Keychain 中安装新证书，如下所示：

    ![][13]

    > [WACOM.NOTE] 证书中的名称可能不同，但将以 **Apple Development iOS Push Notification Services:** 作为前缀。

稍后，你将要使用此证书生成一个 .p12 文件，并将其上载到移动服务以使用 APNS 启用身份验证。

## <a name="profile"></a><span class="short-header">设置应用程序</span>为应用程序创建设置配置文件

1.  返回 [iOS 设置门户][iOS 设置门户]，选择“Provisioning Profiles”（设置配置文件），选择“All”（全部），然后单击“+”按钮创建一个新的配置文件。这将显示“Add iOS Provisiong Profile”（添加 iOS 设置配置文件）向导。

    ![][14]

2.  选择“Development”（开发）下的“iOS App Development”（iOS 应用程序开发）作为设置配置文件类型，然后单击“Continue”（继续）。

    ![][15]

3.  接下来，从“App ID”（应用程序 ID）下拉列表中选择移动服务快速入门应用程序的应用程序 ID，然后单击“Continue”（继续）。

    ![][16]

4.  在“Select certificates”（选择证书）屏幕中，选择前面创建的证书，然后单击“Continue”（继续）。

    ![][17]

5.  接下来，选择要用于测试的“Devices”（设备），然后单击“Continue”（继续）。

    ![][18]

6.  最后，在“Profile Name”（配置文件名称）中为配置文件选择一个名称，单击“Generate”（生成），然后单击“Done”（完成）。

    ![][19]

    ![][20]

    此操作可创建新的配置文件。

7.  在 Xcode 中，打开“Organizer”（组织者）并选择“Devices”（设备）视图，在左窗格的“Library”（库）部分选择“Provisioning Profiles”（设置配置文件），然后导入刚刚创建的设置配置文件。

8.  在左侧，选择你的设备，再次导入该设置配置文件。

9.  在 Keychain Access 中，右键单击新证书，单击“Export”（导出），为证书键入新名称，选择“.p12”格式，然后单击“Save”（保存）。

    ![][21]

    记下文件名和导出证书的位置。

## <a name="configure-hub"></a><span class="short-header">配置通知中心</span>配置通知中心

1.  登录到 [Azure 管理门户][Azure 管理门户]，然后单击屏幕底部的“+新建”。

2.  依次单击“应用程序服务”、“Service Bus”、“通知中心”和“快速创建”。

    ![][22]

3.  键入通知中心的名称，选择所需的区域，然后单击“创建新的通知中心”。

    ![][23]

4.  单击刚刚创建的命名空间（通常为***通知中心名称*-ns**），然后单击顶部的“配置”选项卡。

    ![][24]

5.  单击顶部的“通知中心”选项卡，然后单击你刚刚创建的通知中心。

    ![][25]

6.  选择顶部的“配置”选项卡，然后单击 Apple 通知设置对应的“上载”。然后选择以前导出的“.p12”证书，输入证书的密码。确保选择是要使用“生产”（如果要将推送通知发送到从商店购买你的应用程序的用户）还是“沙盒”（在开发期间）推送服务。

    ![][26]

7.  单击顶部的“仪表板”选项卡，然后单击“连接信息”。记下两个连接字符串。

    ![][27]

你的通知中心现在已配置为使用 APN，并且你有连接字符串用于注册你的应用程序和发送推送通知。

## <a name="connecting-app"></a><span class="short-header">连接应用程序</span>将应用程序连接到通知中心

1.  在 XCode 中，创建新的 iOS 项目，然后选择“单视图应用程序”模板。

    ![][28]

2.  在“目标”下，单击你的项目名称，展开“代码签名标识”，然后在“调试”下选择代码签名标识配置文件。另外，如果在使用更新版本的 XCode，请将“级别”从“基本”切换到“全部”，并将“设置配置文件”行项设置为设置配置文件。

    ![][29]

3.  下载 Azure Mobile SDK for iOS。打开 .zip 文件并将文件夹 WindowsAzureMessaging.framework 拖到 XCode 项目中的 Framework 文件夹。选择“将项目复制到目标组的文件夹中”，然后单击“确定”。

    ![][30]

4.  在 AppDelegate.h 文件中，添加以下导入指令：

         #import <WindowsAzureMessaging/WindowsAzureMessaging.h>

5.  在 AppDelegate.m 文件中，将以下代码添加到`didFinishLaunchingWithOptions` method:

         [[UIApplication sharedApplication] registerForRemoteNotificationTypes: UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];

6.  在同一文件中，添加以下方法：

        - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *) deviceToken {    
            SBNotificationHub* hub = [[SBNotificationHub alloc] initWithConnectionString:
                                      @"<connection string>" notificationHubPath:@"mynh"];

            [hub registerNativeWithDeviceToken:deviceToken tags:nil completion:^(NSError* error) {
                if (error != nil) {
                    NSLog(@"Error registering for notifications: %@", error);
                }
            }];
        }

7.  *（可选）*此外，在同一文件中，如果在应用程序激活时接收通知，则添加以下方法以显示 **UIAlert**：

        - (void)application:(UIApplication *)application didReceiveRemoteNotification: (NSDictionary *)userInfo {
            NSLog(@"%@", userInfo);
            UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Notification" message:
            [[userInfo objectForKey:@"aps"] valueForKey:@"alert"] delegate:nil cancelButtonTitle:
            @"OK" otherButtonTitles:nil, nil];
            [alert show];
        }

8.  在你的设备上运行应用程序。

## <a name="send"></a><span class="short-header">发送通知</span>从后端发送通知

你可以使用通知中心通过 [REST 接口][REST 接口]从任意后端发送通知。在本教程中，我们将使用 .NET 控制台应用程序和移动服务来发送通知，通过节点脚本来执行这些操作。

使用 .NET 应用程序发送通知：

1.  创建新的 Visual C# 控制台应用程序：

    ![][31]

2.  通过使用 [WindowsAzure.ServiceBus NuGet 包][WindowsAzure.ServiceBus NuGet 包]添加对 Azure Service Bus SDK 的引用。在 Visual Studio 主菜单中，依次单击“工具”、“库程序包管理器”和“程序包管理器控制台”。然后，在控制台窗口中键入：

        Install-Package WindowsAzure.ServiceBus

    然后按 Enter。

3.  打开文件 Program.cs 并添加以下 using 语句：

        using Microsoft.ServiceBus.Notifications;

4.  在 `Program` 类中添加以下方法。确保将“中心名称”占位符替换为在门户的“通知中心”选项卡中显示的通知中心名称（如上例中的 **mynotificationhub2**）：

        private static async void SendNotificationAsync()
        {
            NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString("<connection string with full access>", "<hub name>");
            var alert = "{\"aps\":{\"alert\":\"Hello from .NET!\"}}";
            await hub.SendAppleNativeNotificationAsync(alert);
        }

5.  然后将以下行添加到`Main` method:

         SendNotificationAsync();
         Console.ReadLine();

6.  按 F5 键以运行应用程序。你应在设备上收到警报。如果正在使用 Wi-fi，请确保你的连接有效。

可以在 Apple [本地和推送通知编程指南][本地和推送通知编程指南]中查看所有可能的负载。

若要使用移动服务发送通知，请按[移动服务入门][移动服务入门]中的说明操作，然后：

1.  登录到 [Azure 管理门户][Azure 管理门户]并选择你的移动服务。

2.  选择顶部的“计划程序”选项卡。

    ![][32]

3.  创建新的计划作业，插入名称，然后选择“按需”。

    ![][33]

4.  创建作业时，单击该作业名称。然后单击顶部栏上的“脚本”选项卡。

5.  在你的计划程序函数中插入以下脚本。确保将占位符替换为你的通知中心名称和你以前获取的 *DefaultFullSharedAccessSignature* 的连接字符串。单击**“保存”**。

        var azure = require('azure');
        var notificationHubService = azure.createNotificationHubService('<Hubname>', '<SAS Full access >');
        notificationHubService.apns.send(
            null,
            {"aps":
                {
                "alert": "Hello from Mobile Services!"
                }
            },
            function (error)
            {
                if (!error) {
                    console.warn("Notification successful");
                }
            }
        );

6.  单击底部栏上的“运行一次”。你应在设备上收到警报。

## <a name="next-steps"> </a>后续步骤

在这个简单的示例中，你已将通知广播到所有 iOS 设备。为了针对特定用户广播，请参考教程[使用通知中心将通知推送到用户][使用通知中心将通知推送到用户]，如果你要按兴趣细分用户组，请参考[使用通知中心发送突发新闻][使用通知中心发送突发新闻]。在[通知中心指南][通知中心指南]中了解通知中心的详细使用信息。

<!-- Anchors. -->  

<!-- URLs. -->

  [Windows 应用商店 C\#]: /en-us/documentation/articles/notification-hubs-windows-store-dotnet-get-started/ "Windows 应用商店 C#"
  [Windows Phone]: /en-us/documentation/articles/notification-hubs-windows-phone-get-started/ "Windows Phone"
  [iOS]: /en-us/documentation/articles/notification-hubs-ios-get-started/ "iOS"
  [Android]: /en-us/documentation/articles/notification-hubs-android-get-started/ "Android"
  [Kindle]: /en-us/documentation/articles/notification-hubs-kindle-get-started/ "Kindle"
  [Xamarin.iOS]: /en-us/documentation/articles/partner-xamarin-notification-hubs-ios-get-started/ "Xamarin.iOS"
  [Xamarin.Android]: /en-us/documentation/articles/partner-xamarin-notification-hubs-android-get-started/ "Xamarin.Android"
  [生成证书签名请求]: #certificates
  [注册应用程序和启用推送通知]: #register
  [为应用程序创建配置文件]: #profile
  [配置通知中心]: #configure-hub
  [将你的应用程序连接到通知中心]: #connecting-app
  [从后端发送通知]: #send
  [移动服务 iOS SDK]: http://go.microsoft.com/fwLink/?LinkID=266533
  [XCode 4.5]: https://go.microsoft.com/fwLink/p/?LinkID=266532
  [Azure 免费试用]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/

<!-- Images. -->
  []: ./media/notification-hubs-ios-get-started/mobile-services-ios-push-step5.png
  [1]: ./media/notification-hubs-ios-get-started/mobile-services-ios-push-step6.png
  [2]: ./media/notification-hubs-ios-get-started/mobile-services-ios-push-step7.png
  [iOS 设置门户]: http://go.microsoft.com/fwlink/p/?LinkId=272456
  [3]: ./media/notification-hubs-ios-get-started/notification-hub-clone-mobile-services-ios-push-02.png
  [4]: ./media/notification-hubs-ios-get-started/notification-hub-clone-mobile-services-ios-push-03.png
  [5]: ./media/notification-hubs-ios-get-started/notification-hub-clone-mobile-services-ios-push-04.png
  [6]: ./media/notification-hubs-ios-get-started/notification-hub-clone-mobile-services-ios-push-05.png
  [7]: ./media/notification-hubs-ios-get-started/notification-hub-clone-mobile-services-ios-push-06.png
  [8]: ./media/notification-hubs-ios-get-started/notification-hub-clone-mobile-services-ios-push-07.png
  [9]: ./media/notification-hubs-ios-get-started/notification-hub-clone-mobile-services-ios-push-08.png
  [10]: ./media/notification-hubs-ios-get-started/notification-hub-clone-mobile-services-ios-push-10.png
  [11]: ./media/notification-hubs-ios-get-started/notification-hub-clone-mobile-services-ios-push-11.png
  [12]: ./media/notification-hubs-ios-get-started/notification-hub-clone-mobile-services-ios-push-step9.png
  [13]: ./media/notification-hubs-ios-get-started/notification-hub-clone-mobile-services-ios-push-step10.png
  [14]: ./media/notification-hubs-ios-get-started/mobile-services-ios-push-20.png
  [15]: ./media/notification-hubs-ios-get-started/mobile-services-ios-push-21.png
  [16]: ./media/notification-hubs-ios-get-started/mobile-services-ios-push-22.png
  [17]: ./media/notification-hubs-ios-get-started/mobile-services-ios-push-23.png
  [18]: ./media/notification-hubs-ios-get-started/mobile-services-ios-push-24.png
  [19]: ./media/notification-hubs-ios-get-started/mobile-services-ios-push-25.png
  [20]: ./media/notification-hubs-ios-get-started/mobile-services-ios-push-26.png
  [21]: ./media/notification-hubs-ios-get-started/mobile-services-ios-push-step18.png
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [22]: ./media/notification-hubs-ios-get-started/notification-hub-create-from-portal.png
  [23]: ./media/notification-hubs-ios-get-started/notification-hub-create-from-portal2.png
  [24]: ./media/notification-hubs-ios-get-started/notification-hub-select-from-portal.png
  [25]: ./media/notification-hubs-ios-get-started/notification-hub-select-from-portal2.png
  [26]: ./media/notification-hubs-ios-get-started/notification-hub-configure-ios.png
  [27]: ./media/notification-hubs-ios-get-started/notification-hub-connection-strings.png
  [28]: ./media/notification-hubs-ios-get-started/notification-hub-create-ios-app.png
  [29]: ./media/notification-hubs-ios-get-started/notification-hub-create-ios-app2.png
  [30]: ./media/notification-hubs-ios-get-started/notification-hub-create-ios-app3.png
  [REST 接口]: http://msdn.microsoft.com/en-us/library/windowsazure/dn223264.aspx
  [31]: ./media/notification-hubs-ios-get-started/notification-hub-create-console-app.png
  [WindowsAzure.ServiceBus NuGet 包]: http://nuget.org/packages/WindowsAzure.ServiceBus/
  [本地和推送通知编程指南]: http://developer.apple.com/library/mac/#documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html#//apple_ref/doc/uid/TP40008194-CH100-SW1
  [移动服务入门]: /en-us/develop/mobile/tutorials/get-started-ios
  [32]: ./media/notification-hubs-ios-get-started/notification-hub-scheduler1.png
  [33]: ./media/notification-hubs-ios-get-started/notification-hub-scheduler2.png
  [使用通知中心将通知推送到用户]: /en-us/manage/services/notification-hubs/notify-users-aspnet
  [使用通知中心发送突发新闻]: /en-us/manage/services/notification-hubs/breaking-news-dotnet
  [通知中心指南]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx
