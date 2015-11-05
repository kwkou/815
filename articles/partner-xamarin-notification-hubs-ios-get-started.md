<properties
	pageTitle="通知中心入门（Xamarin iOS 应用）| Microsoft Azure"
	description="在本教程中，你将了解如何使用 Azure 通知中心将推送通知发送到 Xamarin iOS 应用程序。"
	services="notification-hubs"
	documentationCenter="xamarin"
	authors="ysxu"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="notification-hubs"
	ms.date="07/28/2015"
	wacn.date="11/02/2015"/>

# 通知中心入门

[AZURE.INCLUDE [notification-hubs-selector-get-started](../includes/notification-hubs-selector-get-started.md)]

##概述
本主题演示如何使用 Azure 通知中心将推送通知发送到 iOS 应用程序。
在本教程中，你将创建一个空白 Xamarin.iOS 应用，它使用 Apple 推送通知服务 (APNs) 接收推送通知。完成后，你将能使用通知中心将推送通知广播到运行你的应用程序的所有设备。[NotificationHubs][GitHub] 应用程序示例中提供了完成的代码。

本教程演示使用通知中心的简单广播方案。

##先决条件

本教程需要满足以下前提条件：

+ [XCode 6.0][Install Xcode]
+ 支持 iOS 7.0（或更高版本）的设备
+ iOS 开发人员计划成员身份
+ [Xamarin.iOS]
+ [Azure 移动服务组件]

   >[AZURE.NOTE]由于推送通知配置要求，你必须在支持 iOS 的设备（iPhone 或 iPad），而不是在模拟器上部署和测试推送通知。

只有在完成本教程后，才能完成有关 Xamarin.iOS 应用程序通知中心的其他所有教程。

> [AZURE.IMPORTANT]若要完成本教程，你必须有一个有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用](/pricing/1rmb-trial/)。

Apple 推送通知服务 (APNS) 使用证书来验证你的移动服务。按照以下说明创建必要的证书并将其上载到你的移动服务。有关正式的 APNS 功能文档，请参阅 [Apple 推送通知服务]。


##<a name="certificates"></a>生成证书签名请求文件

首先，你必须生成证书签名请求 (CSR) 文件，Apple 将使用该文件生成签名证书。

1. 从 Utilities 文件夹中，运行 Keychain Access 工具。

2. 单击“Keychain Access”，展开“Certificate Assistant”（证书助理），然后单击“Request a Certificate from a Certificate Authority...”（从证书颁发机构请求证书...）。

  	![][5]

3. 选择你的“User Email Address”（用户电子邮件地址），键入“Common Name”（公用名）和“CA Email Address”（CA 电子邮件地址）值，确保选中“Saved to disk”（保存到磁盘），然后单击“Continue”（继续）。

  	![][6]

4. 在“Save As”（另存为）中为证书签名请求 (CSR) 文件键入一个名称，在“Where”（位置）中选择一个位置，然后单击“Save”（保存）。

  	![][7]
  
  	此操作会将 CSR 文件保存到选定位置；默认位置是桌面。请记住为此文件选择的位置。

接下来，你将向 Apple 注册你的应用程序、启用推送通知并上载此导出的 CSR 以创建一个推送证书。

##<a name="register"></a>为推送通知注册应用程序

若要将推送通知从移动服务发送到 iOS 应用程序，你必须向 Apple 注册应用程序，还要注册推送通知。

1. 如果你尚未注册应用程序，请导航到 Apple 开发人员中心的 <a href="http://go.microsoft.com/fwlink/p/?LinkId=272456" target="_blank">iOS 设置门户</a>，使用 Apple ID 登录，单击“Identifiers”（标识符），然后单击“App IDs”（应用程序 ID），最后单击“+”符号以注册新的应用程序。

   	![][105]

2. 在“Description”（说明）中为应用程序键入一个名称，在“Bundle Identifier”（捆绑标识符）中输入值，在“App Services”（应用程序服务）部分中选中“Push Notifications”（推送通知）选项，然后单击“Continue”（继续）。

   	![][106]

   	![][107]

   	![][108]
   

	这将生成你的应用程序 ID 并且请求你提交该信息。单击“提交”。
   
   	![][109]
   
	单击“Submit”（提交）后，你将会看到如下所示的“Registration complete”（注册已完成）屏幕。单击“Done”（完成）。
   
   	![][110]

	> [AZURE.NOTE]如果你选择提供“Bundle Identifier”（捆绑标识符）值，而不是 MobileServices.Quickstart，则还必须更新 Xcode 项目中的捆绑标识符值。

3. 找到你刚刚创建的应用程序 ID，然后单击其行。

   	![][111]
   
	单击应用程序 ID 将显示有关应用程序和应用程序 ID 的详细信息：
   
   	![][112]
   
   	![][113]

4. 单击“Edit”（编辑），然后滚动到屏幕底部并单击“Development Push SSL Certificate”（开发推送 SSL 证书）部分下的“Create Certificate...”（创建证书...）。

   	![][114]

	将显示“Add iOS Certificate”（添加 iOS 证书）助手。
   
   	![][115]

	> [AZURE.NOTE]本教程使用开发证书。注册生产证书时使用相同的过程。将证书上载至移动服务时，只需确保设置了相同的证书类型即可。

5. 单击“Choose File”（选择文件），浏览到你在第一个任务中创建的 CSR 文件保存到的位置，然后单击“Generate”（生成）。

  	![][116]
  
6. 门户创建证书之后，单击“Download”（下载），然后单击“Done”（完成）。
 
  	![][118]

  	![][119]
  
   	随后将会下载签名证书并将其保存到计算机上的 **Downloads** 文件夹。

  	![][9]

    > [AZURE.NOTE]默认情况下，下载的文件（开发证书）名为 **aps\_development.cer**。

7. 双击下载的推送证书 **aps\_development.cer**。

	将在 Keychain 中安装新证书，如下所示：

   	![][10]

	> [WACN.NOTE] 证书中的名称可能不同，但将以 <strong>Apple Development iOS Push Notification Services:</strong> 作为前缀。

	接下来，你将使用此证书生成一个 .p12 文件，并将其上载到通知中心以通过 APNS 启用推送通知。

##<a name="profile"></a>为应用程序创建配置文件

1. 返回 <a href="http://go.microsoft.com/fwlink/p/?LinkId=272456" target="_blank">iOS 设置门户</a>，选择“Provisioning Profiles”（设置配置文件），选择“All”（全部），然后单击“+”按钮创建一个新的配置文件。这将显示“Add iOS Provisiong Profile”（添加 iOS 预配配置文件）向导。

   	![][120]

2. 选择“Development”（开发）下的“iOS App Development”（iOS 应用程序开发）作为预配配置文件类型，然后单击“Continue”（继续）。

   	![][121]

3. 接下来，从“App ID”（应用程序 ID）下拉列表中选择移动服务快速入门应用程序的应用程序 ID，然后单击“Continue”（继续）。

   	![][122]

4. 在“Select certificates”（选择证书）屏幕中，选择前面创建的证书，然后单击“Continue”（继续）。
  
   	![][123]

5. 接下来，选择要用于测试的“Devices”（设备），然后单击“Continue”（继续）。

   	![][124]

6. 最后，在“Profile Name”（配置文件名称）中为配置文件选择一个名称，单击“Generate”（生成），然后单击“Done”（完成）。

   	![][125]
   
   	![][126]
	
  	此操作可创建新的配置文件。

7. 在 Xcode 中，打开“Organizer”（组织者）并选择“Devices”（设备）视图，在左窗格的“Library”（库）部分选择“Provisioning Profiles”（预配配置文件），然后导入刚刚创建的预配配置文件。

8. 在左侧，选择你的设备，再次导入该设置配置文件。

9. 在 Keychain Access 中，右键单击新证书，单击“Export”（导出），为证书键入新名称，选择“.p12”格式，然后单击“Save”（保存）。

   	![][18]

  	记下文件名和导出的证书的位置。

这可以确保 Xcode 项目使用新配置文件进行代码签名。接下来，你必须将证书上载至通知中心。

##<a name="configure-hub"></a>配置通知中心

1. 登录到 [Azure 管理门户]，然后单击屏幕底部的“+新建”。

2. 依次单击“应用程序服务”、“服务总线”、“通知中心”、“快速创建”。

   	![][27]

3. 键入通知中心的名称，选择所需的区域，然后单击“创建新的通知中心”。

   	![][28]

4. 单击刚刚创建的命名空间（通常为***通知中心名称*-ns**），然后单击顶部的“配置”选项卡。

   	![][29]

5. 单击顶部的“通知中心”选项卡，然后单击你刚刚创建的通知中心。

   	![][210]

6. 选择顶部的“配置”选项卡，然后单击 Apple 通知设置对应的“上载”。然后选择前面导出的 **.p12** 证书以及证书的密码。确保选择是要使用“生产”（如果要将推送通知发送到从商店购买你的应用程序的用户）还是“沙盒”（在开发期间）推送服务。

   	![][211]

7. 单击顶部的“仪表板”选项卡，然后单击“连接信息”。记下两个连接字符串。

   	![][212]

你的通知中心现在已配置为使用 APN，并且你有连接字符串用于注册你的应用程序和发送推送通知。

##<a name="connecting-app"></a>将应用连接到通知中心

### 下载 WindowsAzure.Messaging 库

使用此程序集可以方便地注册到 Azure 通知中心。可以参考以下说明下载该程序集，也可以在[示例下载][GitHub]中找到其下载链接。

1. 从 GitHub 下载 [WindowsAzure.Messaging] 的源代码。

2. 编译该项目并找到输出程序集 **WindowsAzure.Messaging.dll** - 稍后在设置下面的 Xamarin.iOS 应用程序时需要用到该程序集。


### 创建新项目

1. 在 Xamarin Studio 中，创建新的 iOS 项目，然后选择“统一 API”>“单视图应用程序”模板。

   	![][31]

2. 首先，请添加对 Azure 消息传送组件的引用。在“解决方案”视图中，右键单击你项目的“Components”文件夹，然后选择“获取更多组件”。搜索“Azure 消息传送”组件，并向你的项目添加该组件。

3. 在 **AppDelegate.cs** 中，添加以下 using 语句：

    using WindowsAzure.Messaging;

4. 声明 **SBNotificationHub** 的实例：

		private SBNotificationHub Hub { get; set; }

5. 使用以下变量创建 **Constants.cs** 类：

        // Azure app specific connection string and hub path
        public const string ConnectionString = "<Azure connection string>";
        public const string NotificationHubPath = "<Azure hub path>";


6. 在 **AppDelegate.cs** 中，更新 **FinishedLaunching()** 以匹配以下内容：

        public override bool FinishedLaunching(UIApplication application, NSDictionary launchOptions)
        {
            UIRemoteNotificationType notificationTypes = UIRemoteNotificationType.Alert |
                UIRemoteNotificationType.Badge | UIRemoteNotificationType.Sound;
            UIApplication.SharedApplication.RegisterForRemoteNotificationTypes(notificationTypes);

            return true;
        }

7. 重写 **AppDelegate.cs** 中的 **RegisteredForRemoteNotifications()** 方法：

        public override void RegisteredForRemoteNotifications(UIApplication application, NSData deviceToken)
        {
            Hub = new SBNotificationHub(Constants.ConnectionString, Constants.NotificationHubPath);

            Hub.UnregisterAllAsync (deviceToken, (error) => {
                if (error != null)
                {
                    Console.WriteLine("Error calling Unregister: {0}", error.ToString());
                    return;
                }

                NSSet tags = null; // create tags if you want
                Hub.RegisterNativeAsync(deviceToken, tags, (errorCallback) => {
                    if (errorCallback != null)
                        Console.WriteLine("RegisterNativeAsync error: " + errorCallback.ToString());
                });
            });
        }

8. 重写 **AppDelegate.cs** 中的 **ReceivedRemoteNotification()** 方法：

        public override void ReceivedRemoteNotification(UIApplication application, NSDictionary userInfo)
        {
            ProcessNotification(userInfo, false);
        }

9. 在 **AppDelegate.cs** 中创建以下 **ProcessNotification()** 方法：

        void ProcessNotification(NSDictionary options, bool fromFinishedLaunching)
        {
            // Check to see if the dictionary has the aps key.  This is the notification payload you would have sent
            if (null != options && options.ContainsKey(new NSString("aps")))
            {
                //Get the aps dictionary
                NSDictionary aps = options.ObjectForKey(new NSString("aps")) as NSDictionary;

                string alert = string.Empty;

                //Extract the alert text
                // NOTE: If you're using the simple alert by just specifying
                // "  aps:{alert:"alert msg here"}  " this will work fine.
                // But if you're using a complex alert with Localization keys, etc.,
                // your "alert" object from the aps dictionary will be another NSDictionary.
                // Basically the json gets dumped right into a NSDictionary,
                // so keep that in mind.
                if (aps.ContainsKey(new NSString("alert")))
                    alert = (aps [new NSString("alert")] as NSString).ToString();

                //If this came from the ReceivedRemoteNotification while the app was running,
                // we of course need to manually process things like the sound, badge, and alert.
                if (!fromFinishedLaunching)
                {
                    //Manually show an alert
                    if (!string.IsNullOrEmpty(alert))
                    {
                        UIAlertView avAlert = new UIAlertView("Notification", alert, null, "OK", null);
                        avAlert.Show();
                    }
                }
            }
        }

    > [AZURE.NOTE]你可以选择覆盖 **FailedToRegisterForRemoteNotifications()** 来处理不包括网络连接等的情况。


10. 在你的设备上运行应用程序。

##<a name="send"></a>从后端发送通知

你可以使用通知中心通过 <a href="http://msdn.microsoft.com/zh-cn/library/azure/dn223264.aspx">REST 接口</a>从任意后端发送通知。在本教程中，我们将使用 .NET 控制台应用程序和移动服务来发送通知，通过节点脚本来执行这些操作。

使用 .NET 应用程序发送通知：

1. 创建新的 Visual C# 控制台应用程序： 

   	![][213]

2. 使用 <a href="http://nuget.org/packages/WindowsAzure.ServiceBus/">WindowsAzure.ServiceBus NuGet 包</a>添加对 Azure 服务总线 SDK 的引用。在 Visual Studio 主菜单中，依次单击“工具”、“库包管理器”和“包管理器控制台”。然后，在控制台窗口中键入：

        Install-Package WindowsAzure.ServiceBus and press Enter.

2. 打开文件 Program.cs 并添加以下 using 语句：

        using Microsoft.ServiceBus.Notifications;

3. 在 `Program` 类中添加以下方法。

        private static async void SendNotificationAsync()
        {
            NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString("<connection string with full access>", "<hub name>");
            var alert = "{"aps":{"alert":"Hello from .NET!"}}";
            await hub.SendAppleNativeNotificationAsync(alert);
        }

4. 然后在 `Main` 方法中添加以下行：

         SendNotificationAsync();
		 Console.ReadLine();

5. 按 F5 键以运行应用。你应在设备上收到警报。如果正在使用 Wi-fi，请确保你的连接有效。

可以在 Apple [本地和推送通知编程指南]中查看所有可能的负载。

若要使用移动服务发送通知，请按[移动服务入门]中的说明操作，然后：

1. 登录到[ Azure 管理门户]并选择你的移动服务。

2. 选择顶部的“计划程序”选项卡。

   	![][215]

3. 创建新的计划作业，插入名称，然后选择“按需”。

   	![][216]

4. 创建作业时，单击该作业名称。然后单击顶部栏上的“脚本”选项卡。

5. 在你的计划程序函数中插入以下脚本。确保将占位符替换为你的通知中心名称和你以前获取的 *DefaultFullSharedAccessSignature* 的连接字符串。单击“保存”。

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


6. 单击底部栏上的“运行一次”。你应在设备上收到警报。

## <a name="next-steps"></a>后续步骤

在这个简单的示例中，你已将通知广播到所有 iOS 设备。若要向特定的用户推送消息，请参考教程[使用通知中心将通知推送到用户]，如果要按兴趣组来划分用户，请阅读[使用通知中心发送突发新闻]。请在[通知中心指南]和[适用于 iOS 的通知中心操作方法指南]中了解有关如何使用通知中心的详细信息。

<!-- Anchors. -->
[Generate the certificate signing request]: #certificates
[Register your app and enable push notifications]: #register
[Create a provisioning profile for the app]: #profile
[Configure your Notification Hub]: #configure-hub
[Connecting your app to the Notification Hub]: #connecting-app
[Send notifications from your back-end]: #send
[Next Steps]: #next-steps

<!-- Images. -->
[5]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-step5.png
[6]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-step6.png
[7]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-step7.png

[9]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-step9.png
[10]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-step10.png
[18]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-step18.png
[105]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-05.png
[106]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-06.png
[107]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-07.png
[108]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-08.png
[109]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-09.png
[110]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-10.png
[111]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-11.png
[112]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-12.png
[113]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-13.png
[114]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-14.png
[115]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-15.png
[116]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-16.png

[118]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-18.png
[119]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-19.png

[120]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-20.png
[121]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-21.png
[122]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-22.png
[123]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-23.png
[124]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-24.png
[125]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-25.png
[126]: ./media/partner-xamarin-notification-hubs-ios-get-started/mobile-services-ios-push-26.png

[27]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-create-from-portal.png
[28]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-create-from-portal2.png
[29]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-select-from-portal.png
[210]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-select-from-portal2.png
[211]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-configure-ios.png
[212]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-connection-strings.png


[213]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-create-console-app.png



[215]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-scheduler1.png
[216]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-scheduler2.png


[31]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-create-ios-app.png




<!-- URLs. -->
[移动服务入门]: /documentation/articles/notification-hubs-mobile-services-cross-platform-notify-users
[ Azure 管理门户]: https://manage.windowsazure.cn/
[通知中心指南]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx
[适用于 iOS 的通知中心操作方法指南]: http://msdn.microsoft.com/zh-cn/library/jj927168.aspx
[Install Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[使用通知中心将通知推送到用户]: /documentation/articles/notification-hubs-aspnet-backend-windows-dotnet-notify-users
[使用通知中心发送突发新闻]: /documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news

[本地和推送通知编程指南]: http://developer.apple.com/library/mac/#documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html#//apple_ref/doc/uid/TP40008194-CH100-SW1
[Apple 推送通知服务]: http://go.microsoft.com/fwlink/p/?LinkId=272584

[Azure 移动服务组件]: http://components.xamarin.com/view/azure-mobile-services/
[GitHub]: http://go.microsoft.com/fwlink/p/?LinkId=331329
[WindowsAzure.Messaging]: https://github.com/infosupport/WindowsAzure.Messaging.iOS
