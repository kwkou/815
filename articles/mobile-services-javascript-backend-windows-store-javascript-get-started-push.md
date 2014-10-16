<properties pageTitle="Get started with push notifications (Windows Store) | Mobile Dev Center" metaKeywords="" description="Learn how to use Azure Mobile Services and Notification Hubs to send push notifications to your Windows Store app." metaCanonical="" services="mobile" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services" authors="glenga" solutions="" manager="" editor="" />

# 移动服务中的推送通知入门

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push" title="Windows Store C#">Windows 应用商店 C\#</a><a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-get-started-push" title="Windows Store JavaScript" class="current">Windows 应用商店 JavaScript</a><a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push" title="Windows Phone">Windows Phone</a><a href="/zh-cn/documentation/articles/mobile-services-ios-get-started-push" title="iOS">iOS</a><a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-android-get-started-push" title="Android">Android</a></div>

<div class="dev-center-tutorial-subselector"><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-push/" title=".NET backend">.NET 后端</a> | <a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-get-started-push/"  title="JavaScript backend" class="current">JavaScript 后端</a></div>

本主题说明如何使用 Azure 移动服务向 Windows 应用商店应用程序发送推送通知。
在本教程中，你将要使用 Azure 通知中心为快速入门项目启用推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会使用通知中心发送一条推送通知。创建的通知中心可在移动服务中任意使用，可独立于移动服务进行管理，并可供其他应用程序和服务使用。

> [WACOM.NOTE] 本教程演示了移动服务与通知中心的集成功能，该功能当前以预览版提供。默认情况下，未在 JavaScript 后端中启用通过通知中心发送推送通知的功能。创建新的通知中心后，集成过程将不可逆。

本教程将指导你完成启用推送通知的以下基本步骤：

1.  [将应用程序注册到 WNS 并配置移动服务][]
2.  [更新应用程序以注册通知][]
3.  [更新服务器脚本以发送推送通知][]
4.  [插入数据以接收推送通知][]

本教程基于移动服务快速入门。在开始学习本教程之前，必须先完成[移动服务入门][]或[数据处理入门][]，以将项目连接到移动服务。未连接移动服务时，“添加推送通知”向导将为你创建此连接。

<a id="register"></a>
## 将应用程序注册到 WNS 并配置移动服务

[WACOM.INCLUDE [mobile-services-javascript-backend-register-windows-store-app][]]

现在，你的移动服务和应用程序都已配置为使用 WNS 和通知中心。接下来，你要更新 Windows 应用商店应用程序，以注册通知。

<a id="update-app"></a>
## 更新应用程序以注册通知

只有在你注册通知通道后，你的应用程序才能接收推送通知。

1.  打开文件 default.js，并在创建 "MobileServiceClient" 实例的代码后面插入以下代码：

        // Request a push notification channel.
        Windows.Networking.PushNotifications
        .PushNotificationChannelManager
        .createPushNotificationChannelForApplicationAsync()
        .then(function (channel) {
        // Register for notifications using the new channel
        client.push.registerNative(channel.uri);                    
            });      

        此代码从 WNS 检索应用程序的 ChannelURI，然后注册该 ChannelURI，以便将其用于推送通知。

2.  按 "F5" 键以运行应用程序。将显示包含注册密钥的弹出式对话框。

3.  打开 Package.appxmanifest 文件，并确保“应用程序 UI”选项卡上的“支持 Toast 通知”已设置为“是” 。

    ![][]

    这可以确保你的应用程序能够引发 toast 通知。

<a id="update-scripts"></a>
## 更新服务器脚本以发送推送通知

[WACOM.INCLUDE [mobile-services-javascript-update-script-notification-hubs][]]

<a id="test"></a>
## 在应用程序中测试推送通知

1.  在 Visual Studio 中，按 F5 键运行应用程序。

2.  在应用程序中的“插入 TodoItem”内键入文本，然后单击“保存” 。

    ![][1]

    请注意，完成插入后，应用程序将会接收来自 WNS 的推送通知。

    ![][2]

<a name="next-steps"> </a>
## 后续步骤

本教程演示了有关如何使 Windows 应用商店应用程序处理移动服务中的数据的基础知识。接下来，建议你完成下列教程之一，这些教程是基于本教程中创建的 GetStartedWithData 应用程序制作的：

-   [通知中心入门][]
    了解如何在 Windows 应用商店应用程序中利用通知中心。

-   [向订户发送通知][]
    了解用户如何注册和接收他们感兴趣的类别的推送通知。

-   [向用户发送通知][]
    了解如何从移动服务向任一设备上的特定用户发送推送通知。

-   [向用户发送跨平台通知][]
    了解如何使用模板从移动服务发送推送通知，且不会在后端中产生平台特定的负载。

建议你了解有关以下移动服务主题的详细信息：

-   [数据处理入门][]
    了解有关使用移动服务存储和查询数据的详细信息。

-   [身份验证入门][]
    了解如何使用 Windows 帐户对应用程序用户进行身份验证。

-   [移动服务服务器脚本参考][]
    了解有关注册和使用服务器脚本的详细信息。

-   [移动服务 .NET 操作方法概念性参考][]
    了解有关如何将移动服务与 .NET 一起使用的详细信息。

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-get-started-push "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/mobile-services-ios-get-started-push "iOS"
  [Android]: /zh-cn/documentation/articles/mobile-services-javascript-backend-android-get-started-push "Android"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-push/ ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-get-started-push/ "JavaScript 后端"
  [将应用程序注册到 WNS 并配置移动服务]: #register
  [更新应用程序以注册通知]: #update-app
  [更新服务器脚本以发送推送通知]: #update-scripts
  [插入数据以接收推送通知]: #test
  [移动服务入门]: /zh-cn/documentation/articles/mobile-services-windows-store-get-started
  [数据处理入门]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-data
  [mobile-services-javascript-backend-register-windows-store-app]: ../includes/mobile-services-javascript-backend-register-windows-store-app.md
  [0]: ./media/mobile-services-windows-store-javascript-get-started-push-vs2012/mobile-app-enable-toast-win8.png
  [mobile-services-javascript-update-script-notification-hubs]: ../includes/mobile-services-javascript-update-script-notification-hubs.md
  [1]: ./media/mobile-services-windows-store-javascript-get-started-push/mobile-quickstart-push1.png
  [2]: ./media/mobile-services-windows-store-javascript-get-started-push/mobile-quickstart-push2.png
  [通知中心入门]: /zh-cn/manage/services/notification-hubs/getting-started-windows-dotnet/
  [向订户发送通知]: /zh-cn/manage/services/notification-hubs/breaking-news-dotnet/
  [向用户发送通知]: /zh-cn/manage/services/notification-hubs/notify-users/
  [向用户发送跨平台通知]: /zh-cn/manage/services/notification-hubs/notify-users-xplat-mobile-services/
  [身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-users
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/?LinkId=262293
  [移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-html-how-to-use-client-library
