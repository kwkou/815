<properties pageTitle="推送通知入门（Windows 应用商店）| 移动开发人员中心" metaKeywords="" description="了解如何使用 Azure 移动服务和通知中心将推送通知发送到您的 Windows 应用商店应用程序。" metaCanonical="" services="mobile" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services" authors="glenga" solutions="" manager="" editor="" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-windows-store" ms.devlang="javascript" ms.topic="article" ms.date="09/23/2014" ms.author="glenga" />


# 向移动服务应用程序添加推送通知

[WACOM.INCLUDE [mobile-services-selector-get-started-push-legacy](../includes/mobile-services-selector-get-started-push-legacy.md)]

本主题说明如何使用 Azure 移动服务向 Windows 应用商店应用程序发送推送通知。 
在本教程中，你将要使用 Azure 通知中心为快速入门项目启用推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会使用通知中心发送一条推送通知。创建的通知中心可在移动服务中任意使用，可独立于移动服务进行管理，并可供其他应用程序和服务使用。

>[WACOM.NOTE]本主题演示了如何使用面向 Windows 应用商店应用程序的 Windows 通知服务 (WNS) 手动配置推送通知。您可以使用 Visual Studio 2013 工具在 Windows 应用商店应用程序项目中自动配置相同的推送通知。有关详细信息，请参阅本教程的[通用 Windows 应用程序版本](/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-get-started-push)部分。

本教程将指导你完成启用推送通知的以下基本步骤：

1. [将应用程序注册到 WNS 并配置移动服务](#register)
2. [更新应用程序以注册通知](#update-app)
3. [更新服务器脚本以发送推送通知](#update-scripts)
3. [插入数据以接收推送通知](#test)

本教程基于移动服务快速入门。在开始学习本教程之前，必须先完成[移动服务入门]或[数据处理入门]，以将项目连接到移动服务。如果尚未连接移动服务，"添加推送通知"向导将为你创建此连接。 

##<a id="register"></a> 将应用程序注册到 WNS 并配置移动服务

[WACOM.INCLUDE [mobile-services-notification-hubs-register-windows-store-app](../includes/mobile-services-notification-hubs-register-windows-store-app.md)]

现在，你的移动服务和应用程序都已配置为使用 GCM 和通知中心。接下来，你要更新 Windows 应用商店应用程序，以注册通知。

##<a id="update-app"></a> 更新应用程序以注册通知

只有在你注册通知通道后，你的应用程序才能接收推送通知。

1. 打开文件 default.js，并在创建 **MobileServiceClient** 实例的代码后面插入以下代码：

        // Request a push notification channel.
        Windows.Networking.PushNotifications
            .PushNotificationChannelManager
            .createPushNotificationChannelForApplicationAsync()
            .then(function (channel) {
                // Register for notifications using the new channel
                client.push.registerNative(channel.uri);                    
            });      

	此代码从 WNS 检索应用程序的 ChannelURI，然后注册该 ChannelURI，以便将其用于推送通知。

2. 按 **F5** 键以运行应用。将显示包含注册密钥的弹出式对话框。

6. 打开 Package.appxmanifest 文件，并确保**应用程序 UI**选项卡上的**支持 Toast 通知**已设置为**是**。

   ![][2]

   这可以确保你的应用程序能够引发 toast 通知。 

##<a id="update-scripts"></a> 更新服务器脚本以发送推送通知

[WACOM.INCLUDE [mobile-services-javascript-update-script-notification-hubs](../includes/mobile-services-javascript-update-script-notification-hubs.md)]

##<a id="test"></a> 在应用程序中测试推送通知

[WACOM.INCLUDE [mobile-services-windows-store-test-push](../includes/mobile-services-windows-store-test-push.md)]

## <a name="next-steps"> </a>后续步骤

本教程演示了启用 Windows 应用商店应用程序以便使用移动服务和通知中心发送推送通知的基础知识。接下来，请考虑完成下一个教程，[向已经过身份验证的用户发送推送通知]，它显示如何使用标记将推送通知从移动服务发送到只有经过身份验证的用户。

<!---+ [向经过身份验证的用户发送推送通知]
	<br/>了解如何使用标记将推送通知从移动服务发送到只有经过身份验证的用户。

+ [将广播通知发送到订阅用户]
	<br/>了解用户如何注册并接收其感兴趣的类别的推送通知。

+ [将基于模板的通知发送到订阅用户]
	<br/>了解如何使用模板通过移动服务发送推送通知，而无需在后端处理特定于平台的负载。
-->

下面的主题介绍了有关移动服务和通知中心的详细信息：

* [数据处理入门]
  <br/>了解有关使用移动服务存储和查询数据的详细信息。

* [身份验证入门]
  <br/>了解如何通过不同帐户类型（使用移动服务）对应用程序的用户进行身份验证。

* [什么是通知中心？]
  <br/>了解有关通知中心跨所有主要的客户端平台向您的应用程序交付通知的详细信息。

* [调试通知中心应用程序](http://go.microsoft.com/fwlink/p/?linkid=386630)
  </br>获取故障排除和调试通知中心解决方案的指南。 

* [移动服务 .NET 操作方法概念性参考]
  <br/>了解有关如何将移动服务与 .NET 一起使用的详细信息。

* [移动服务服务器脚本参考]
  <br/>了解有关如何在移动服务中实施业务逻辑的详细信息。

<!-- Anchors. -->

<!-- Images. -->


[2]: ./media/mobile-services-javascript-backend-windows-store-javascript-get-started-push/mobile-app-enable-toast-win8.png


<!-- URLs. -->
[提交应用程序页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[我的应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-windows-store-get-started
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-data
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-users

[移动服务服务器脚本参考]: /zh-cn/documentation/articles/mobile-services-how-to-use-server-scripts/
[移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library


[向已经过身份验证的用户发送推送通知]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-push-notifications-app-users/

[什么是通知中心？]: /zh-cn/documentation/articles/notification-hubs-overview/
[将广播通知发送到订阅用户]: /zh-cn/documentation/articles/notification-hubs-windows-store-javascript-send-breaking-news/
[将基于模板的通知发送到订阅用户]: /zh-cn/documentation/articles/notification-hubs-windows-store-javascript-send-localized-breaking-news/
