<properties
	pageTitle="向 Xamarin.Forms 应用添加推送通知 | Azure"
	description="了解如何使用 Azure 服务将多平台推送通知发送到 Xamarin.Forms 应用。"
	services="app-service\mobile"
	documentationCenter="xamarin"
	authors="adrianhall"
	manager="dwrede"
	editor=""/>  


<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-xamarin"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="10/01/2016"
	wacn.date="11/21/2016"
	ms.author="adrianha"/>

# 向 Xamarin.Forms 应用添加推送通知

[AZURE.INCLUDE [app-service-mobile-selector-get-started-push](../../includes/app-service-mobile-selector-get-started-push.md)]

##概述

本教程演示如何使用 Azure 服务将推送通知发送到各种本机设备平台（Android、iOS 和 Windows）上运行的 Xamarin.Forms 应用。使用 Azure 通知中心从 Azure 移动应用后端发送推送通知。使用模板注册，这样可以将同一消息发送到使用不同推送通知服务 (PNS) 的所有平台上运行的设备。有关发送跨平台推送通知的详细信息，请参阅 [Azure 通知中心](/documentation/articles/notification-hubs-templates-cross-platform-push-messages/)文档。

向 Xamarin.Forms 应用支持的每个项目添加推送通知。每次在后端中插入一条记录时，都会发送一条推送通知。

##先决条件

为了使本教程达到最佳效果，建议用户先完成[创建 Xamarin.Forms 应用](/documentation/articles/app-service-mobile-xamarin-forms-get-started/)教程。完成此教程后，用户将获得一个 Xamarin.Forms 项目，它是一个多平台 TodoList 应用。

如果不使用下载的快速入门服务器项目，必须将推送通知扩展包添加到项目。有关服务器扩展包的详细信息，请参阅 [Work with the .NET backend server SDK for Azure Mobile Apps](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/)（使用适用于 Azure 移动应用的 .NET 后端服务器 SDK）。

向 iOS 设备发送推送通知需要 [Apple 开发人员计划成员身份](https://developer.apple.com/programs/ios/)。另外，必须使用物理 iOS 设备，因为 iOS 模拟器不支持推送通知。

##<a name="create-hub"></a>创建通知中心

[AZURE.INCLUDE [app-service-mobile-create-notification-hub](../../includes/app-service-mobile-create-notification-hub.md)]

##更新服务器项目以发送推送通知

[AZURE.INCLUDE [app-service-mobile-update-server-project-for-push-template](../../includes/app-service-mobile-update-server-project-for-push-template.md)]

##（可选）配置和运行 iOS 项目

本部分用于运行适用于 iOS 设备的 Xamarin iOS 项目。如果不使用 iOS 设备，可以跳过本部分。

[AZURE.INCLUDE [notification-hubs-xamarin-enable-apple-push-notifications](../../includes/notification-hubs-xamarin-enable-apple-push-notifications.md)]


####为 APNS 配置通知中心

1. 登录到 [Azure 门户预览](https://portal.azure.cn/)。单击“浏览”>“移动应用”> 移动应用 >“设置”>“推送” >“Apple (APNS)”>“上载证书”。上载先前导出的.p12 推送证书文件。如果已创建用于开发和测试的开发推送证书，请务必选择“沙盒”。否则，请选择“生产”。现在，服务已配置为在 iOS 上使用通知中心。

	![](./media/app-service-mobile-xamarin-ios-get-started-push/mobile-app-upload-apns-cert.png)


	接下来，将在 Xamarin Studio 或 Visual Studio 中配置 iOS 项目设置。

[AZURE.INCLUDE [app-service-mobile-xamarin-ios-configure-project](../../includes/app-service-mobile-xamarin-ios-configure-project.md)]


####向 iOS 应用添加推送通知

1. 在 **iOS** 项目中，打开 AppDelegate.cs，并在代码文件的顶部添加以下 **using** 语句。

        using Newtonsoft.Json.Linq;

4. 在 **AppDelegate** 类中，添加 **RegisteredForRemoteNotifications** 事件的重写以注册通知：

        public override void RegisteredForRemoteNotifications(UIApplication application, 
			NSData deviceToken)
        {
            const string templateBodyAPNS = "{"aps":{"alert":"$(messageParam)"}}";

            JObject templates = new JObject();
            templates["genericMessage"] = new JObject
                {
                  {"body", templateBodyAPNS}
                };

            // Register for push with your mobile app
            Push push = TodoItemManager.DefaultManager.CurrentClient.GetPush();
            push.RegisterAsync(deviceToken, templates);
        }

5. 在 **AppDelegate** 中，还为 **DidReceivedRemoteNotification** 事件处理程序添加以下重写：

        public override void DidReceiveRemoteNotification(UIApplication application, 
			NSDictionary userInfo, Action<UIBackgroundFetchResult> completionHandler)
        {
            NSDictionary aps = userInfo.ObjectForKey(new NSString("aps")) as NSDictionary;

            string alert = string.Empty;
            if (aps.ContainsKey(new NSString("alert")))
                alert = (aps[new NSString("alert")] as NSString).ToString();

            //show alert
            if (!string.IsNullOrEmpty(alert))
            {
                UIAlertView avAlert = new UIAlertView("Notification", alert, null, "OK", null);
                avAlert.Show();
            }
        }

	运行该应用时，此方法可处理传入通知。

2. 在 **AppDelegate** 类中，向 **FinishedLaunching** 方法添加以下代码：

        // Register for push notifications.
        var settings = UIUserNotificationSettings.GetSettingsForTypes(
            UIUserNotificationType.Alert
            | UIUserNotificationType.Badge
            | UIUserNotificationType.Sound,
            new NSSet());

        UIApplication.SharedApplication.RegisterUserNotificationSettings(settings);
        UIApplication.SharedApplication.RegisterForRemoteNotifications();

	这将启用对远程通知的支持并请求推送注册。

你的应用现已更新，可支持推送通知。

####在 iOS 应用中测试推送通知

1. 右键单击 iOS 项目，然后单击“设为启动项目”。

2. 在 Visual Studio 中按“运行”按钮或 **F5** 生成项目并在 iOS 设备中启动应用，然后单击“确定”接受推送通知。

	> [AZURE.NOTE] 你必须显式接受来自应用程序的推送通知。此请求只会在首次运行应用程序时出现。

3. 在应用中，键入一项任务，然后单击加号 (**+**) 图标。

4. 检查是否已收到通知，然后单击“确定”以取消通知。


##（可选）配置和运行 Windows 项目

本部分用于运行适用于 Windows 设备的 Xamarin.Forms WinApp 和 WinPhone81 项目。这些步骤也支持通用 Windows 平台 (UWP) 项目。如果不使用 Windows 设备，可以跳过本部分。


####向 WNS 注册用于推送通知的 Windows 应用

[AZURE.INCLUDE [app-service-mobile-register-wns](../../includes/app-service-mobile-register-wns.md)]


####为 WNS 配置通知中心

[AZURE.INCLUDE [app-service-mobile-configure-wns](../../includes/app-service-mobile-configure-wns.md)]


####向 Windows 应用添加推送通知

1. 在 Visual Studio 中，打开 Windows 项目中的 **App.xaml.cs**，并添加以下 **using** 语句。

		using Newtonsoft.Json.Linq;
		using Microsoft.WindowsAzure.MobileServices;
		using System.Threading.Tasks;
		using Windows.Networking.PushNotifications;
		using <your_TodoItemManager_portable_class_namespace>;

	将 `<your_TodoItemManager_portable_class_namespace>` 替换为包含 `TodoItemManager` 类的可移植项目的命名空间。
 

2. 在 App.xaml.cs 中，添加以下 **InitNotificationsAsync** 方法：

        private async Task InitNotificationsAsync()
        {
            var channel = await PushNotificationChannelManager
                .CreatePushNotificationChannelForApplicationAsync();

            const string templateBodyWNS = 
				"<toast><visual><binding template="ToastText01"><text id="1">$(messageParam)</text></binding></visual></toast>";

            JObject headers = new JObject();
            headers["X-WNS-Type"] = "wns/toast";

            JObject templates = new JObject();
            templates["genericMessage"] = new JObject
			{
				{"body", templateBodyWNS},
				{"headers", headers} // Needed for WNS.
			};

            await TodoItemManager.DefaultManager.CurrentClient.GetPush()
				.RegisterAsync(channel.Uri, templates);
        }

	此方法获取推送通知通道，并注册模板以从通知中心接收模板通知。支持 *messageParam* 的模板通知将传送到此客户端。

3. 在 App.xaml.cs 中，通过添加 `async` 修饰符，然后在方法的末尾添加以下代码行，更新 **OnLaunched** 事件处理程序方法定义：

        await InitNotificationsAsync();

	这可确保每次启动应用时，创建或刷新推送通知注册。请务必执行此操作，以保证 WNS 推送通道始终处于活动状态。

4. 在 Visual Studio 的解决方案资源管理器中，打开 **Package.appxmanifest** 文件，并在“通知”下将“支持 Toast 通知”设置为“是”。

5. 生成该应用并确认没有错误。客户端应用现在应从移动应用后端注册模板通知。对于解决方案中的每个 Windows 项目，重复此部分的操作。


####在 Windows 应用中测试推送通知

1. 在 Visual Studio 中，右键单击 Windows 项目，然后单击“设为启动项目”。

2. 按“运行”按钮生成项目并启动应用程序。

3. 在应用中，为新 todoitem 键入一个名称，然后单击加号 (**+**) 图标以添加它。

4. 确认在添加该项目时收到了通知。

##后续步骤

了解有关推送通知的详细信息：

* [使用适用于 Azure 移动应用的 .NET 后端服务器 SDK](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/#how-to-add-tags-to-a-device-installation-to-enable-push-to-tags)  
标记可用于通过推送定位分段客户。了解如何将标记添加到设备安装。

* [诊断推送通知问题](/documentation/articles/notification-hubs-push-notification-fixer/)  
有多种原因可能导致通知被丢弃或最终未到达设备。本主题演示如何分析和确定推送通知失败的根本原因。

请考虑继续学习以下教程之一：

* [向应用添加身份验证](/documentation/articles/app-service-mobile-xamarin-forms-get-started-users/)  
了解如何使用标识提供者对应用的用户进行身份验证。

* [为应用启用脱机同步](/documentation/articles/app-service-mobile-xamarin-forms-get-started-offline-data/)  
  了解如何使用移动应用后端向应用添加脱机支持。脱机同步允许最终用户与移动应用交互（查看、添加或修改数据），即使在没有网络连接时也是如此。

<!-- Images. -->

<!-- URLs. -->
[Install Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[Xcode]: https://go.microsoft.com/fwLink/?LinkID=266532
[apns object]: http://go.microsoft.com/fwlink/p/?LinkId=272333

<!---HONumber=Mooncake_1114_2016-->