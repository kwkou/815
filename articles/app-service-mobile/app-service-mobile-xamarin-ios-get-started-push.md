<properties
	pageTitle="使用 Azure 应用服务向 Xamarin.iOS 应用添加推送通知"
	description="了解如何使用 Azure 应用服务将推送通知发送到 Xamarin iOS 应用"
	services="app-service\mobile"
	documentationCenter="xamarin"
	authors="wesmc7777"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-xamarin-ios"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="05/05/2016"
	wacn.date="09/26/2016"
	ms.author="adrianha"/>

# 向 Xamarin.iOS 应用添加推送通知

[AZURE.INCLUDE [app-service-mobile-selector-get-started-push](../../includes/app-service-mobile-selector-get-started-push.md)]

##概述

本教程是在必须先完成的 [Xamarin.iOS 快速入门](/documentation/articles/app-service-mobile-xamarin-ios-get-started/)教程的基础之上制作的。将向 Xamarin.iOS 快速入门项目添加推送通知，以便每次插入一条记录时，发送一条推送通知。如果不使用下载的快速入门服务器项目，必须将推送通知扩展包添加到项目。有关服务器扩展包的详细信息，请参阅 [Work with the .NET backend server SDK for Azure Mobile Apps](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/)（使用适用于 Azure 移动应用的 .NET 后端服务器 SDK）。

##先决条件

* 完成 [Xamarin.iOS 快速入门](/documentation/articles/app-service-mobile-xamarin-ios-get-started/)教程。

* 物理 iOS 设备。iOS 模拟器不支持推送通知。

##在 Apple 的开发人员门户上为推送通知注册应用

[AZURE.INCLUDE [通知中心：Xamarin 启用 Apple 推送通知](../../includes/notification-hubs-xamarin-enable-apple-push-notifications.md)]

##配置移动应用以发送推送通知

若要将应用配置为发送通知，请创建一个新中心，并针对要使用的平台通知服务配置它。

1. 在 [Azure 门户预览](https://portal.azure.cn/)中，依次单击“浏览”>“移动应用”> 移动应用 >“设置”>“移动”>“推送”>“通知中心”>“+ 通知中心”，提供新通知中心的名称和命名空间，然后单击“确定”按钮。

	![](./media/app-service-mobile-xamarin-ios-get-started-push/mobile-app-configure-notification-hub.png)

2. 在“创建通知中心”边栏选项卡中，单击“创建”。

3. 单击“推送”>“Apple (APNS)”>“上载证书”。上载先前导出的.p12 推送证书文件。如果已创建用于开发和测试的开发推送证书，请务必选择“沙盒”。否则，请选择“生产”。

	![](./media/app-service-mobile-xamarin-ios-get-started-push/mobile-app-upload-apns-cert.png)

现在，服务已配置为在 iOS 上使用通知中心。

##更新服务器项目以发送推送通知

[AZURE.INCLUDE [app-service-mobile-update-server-project-for-push-template](../../includes/app-service-mobile-update-server-project-for-push-template.md)]

##配置 Xamarin.iOS 项目

[AZURE.INCLUDE [app-service-mobile-xamarin-ios-configure-project](../../includes/app-service-mobile-xamarin-ios-configure-project.md)]

## <a name="add-push"></a>向应用程序添加推送通知

1. 在 **QSTodoService** 中，添加以下属性使 **AppDelegate** 可以获取移动客户端：

            public MobileServiceClient GetClient {
            get
            {
                return client;
            }
            private set
            {
                client = value;
            }
        }

1. 在 **AppDelegate.cs** 文件顶部添加以下 `using` 语句。

		using Microsoft.WindowsAzure.MobileServices;
		using Newtonsoft.Json.Linq;

2. 在 **AppDelegate** 中，重写 **FinishedLaunching** 事件：

        public override bool FinishedLaunching(UIApplication application, NSDictionary launchOptions)
        {
            // registers for push for iOS8
            var settings = UIUserNotificationSettings.GetSettingsForTypes(
                UIUserNotificationType.Alert
                | UIUserNotificationType.Badge
                | UIUserNotificationType.Sound,
                new NSSet());

            UIApplication.SharedApplication.RegisterUserNotificationSettings(settings);
            UIApplication.SharedApplication.RegisterForRemoteNotifications();

            return true;
        }

3. 在同一文件中，重写 **RegisteredForRemoteNotifications** 事件。在此代码中，将注册一个简单的模板通知，服务器会将此通知发送到所有支持的平台。



        public override void RegisteredForRemoteNotifications(UIApplication application, NSData deviceToken)
        {
            MobileServiceClient client = QSTodoService.DefaultService.GetClient;

            const string templateBodyAPNS = "{"aps":{"alert":"$(messageParam)"}}";

            JObject templates = new JObject();
            templates["genericMessage"] = new JObject
            {
                {"body", templateBodyAPNS}
            };

            // Register for push with your mobile app
            var push = client.GetPush();
            push.RegisterAsync(deviceToken, templates);
        }


4. 然后，重写 **DidReceivedRemoteNotification** 事件：

        public override void DidReceiveRemoteNotification (UIApplication application, NSDictionary userInfo, Action<UIBackgroundFetchResult> completionHandler)
        {
            NSDictionary aps = userInfo.ObjectForKey(new NSString("aps")) as NSDictionary;

            string alert = string.Empty;
            if (aps.ContainsKey(new NSString("alert")))
                alert = (aps [new NSString("alert")] as NSString).ToString();

            //show alert
            if (!string.IsNullOrEmpty(alert))
            {
                UIAlertView avAlert = new UIAlertView("Notification", alert, null, "OK", null);
                avAlert.Show();
            }
        }

你的应用现已更新，可支持推送通知。

## <a name="test"></a>在应用程序中测试推送通知

1. 在支持 iOS 的设备中按“运行”按钮生成项目并启动应用，然后单击“确定”接受推送通知。

	> [AZURE.NOTE] 你必须显式接受来自应用程序的推送通知。此请求只会在首次运行应用程序时出现。

2. 在应用中，键入一项任务，然后单击加号 (**+**) 图标。

3. 检查是否已收到通知，然后单击“确定”以取消通知。

4. 重复步骤 2 并立即关闭应用，然后检查是否已显示通知。

你已成功完成本教程。

<!-- Images. -->

<!-- URLs. -->

<!---HONumber=Mooncake_0919_2016-->