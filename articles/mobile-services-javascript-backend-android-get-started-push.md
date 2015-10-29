
<properties
	pageTitle="推送通知入门 (Android JavaScript) | Windows Azure"
	description="了解如何使用 Azure 移动服务向 Android JavaScript 应用程序发送推送通知。"
	services="mobile-services, notification-hubs"
	documentationCenter="android"
	authors="RickSaling"
	writer="ricksal"
	manager="dwrede"
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="06/16/2015" 
	wacn.date="10/22/2015"/>


#  向移动服务 Android 应用程序添加推送通知

[AZURE.INCLUDE [mobile-services-selector-get-started-push](../includes/mobile-services-selector-get-started-push.md)]

##  摘要

本主题说明如何使用 Azure 移动服务将推送通知发送到使用 Google Cloud Messaging（“GCM”）的 Android 应用程序。你将要向本教程必需的快速入门项目添加推送通知。推送通知是使用移动服务随附的 Azure 通知中心启用的。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。




## 先决条件

[AZURE.INCLUDE [mobile-services-android-prerequisites](../includes/mobile-services-android-prerequisites.md)]

## <a id="register"></a>启用 Google Cloud Messaging

[AZURE.INCLUDE [mobile-services-enable-Google-cloud-messaging](../includes/mobile-services-enable-Google-cloud-messaging.md)]

## <a id="configure"></a>配置移动服务以发送推送请求

[AZURE.INCLUDE [mobile-services-android-configure-push](../includes/mobile-services-android-configure-push.md)]

## <a id="add-push"></a>向应用程序添加推送通知



下一步就是安装 Google Play Services。Google Cloud Messaging 对开发和测试提出了一些最低的 API 级别要求，清单中的 **minSdkVersion** 属性必须符合这些要求。

如果你要对某台较旧的设备进行测试，请查阅[设置 Google Play Services SDK]，以确定此值可设置到的最小值，并相应地进行设置。

### 将 Google Play Services 添加到项目

[AZURE.INCLUDE [添加 Play Services](../includes/mobile-services-add-Google-play-services.md)]

### 添加代码

[AZURE.INCLUDE [mobile-services-android-getting-started-with-push](../includes/mobile-services-android-getting-started-with-push.md)]


## <a id="update-scripts"></a>在管理门户中更新已注册的插入脚本

[AZURE.INCLUDE [mobile-services-javascript-backend-android-push-insert-script](../includes/mobile-services-javascript-backend-android-push-insert-script.md)]


## <a id="test"></a>在应用程序中测试推送通知
   
你可以通过以下方式测试应用程序：使用 USB 电缆直接连接 Android 手机，或者在模拟器中使用虚拟设备。

### 设置用于测试的 Android 模拟器

当你在模拟器中运行此应用程序时，请确保使用支持 Google API 的 Android 虚拟设备 (AVD)。

1. 从右侧的工具栏中，选择“Android 虚拟设备管理器”，选择你的设备，然后单击右侧的编辑图标。

	![](./media/mobile-services-javascript-backend-android-get-started-push/mobile-services-android-virtual-device-manager.png)

2. 在设备描述行中选择“更改”，选择“Google API”，然后单击“确定”。

   	![](./media/mobile-services-javascript-backend-android-get-started-push/mobile-services-android-virtual-device-manager-edit.png)

	这样便指定了 AVD 要使用 Google API。

### 运行测试

1. 在“运行”菜单项中，单击“运行应用程序”以启动应用程序。

2. 在应用程序中键入有意义的文本（例如 _A new Mobile Services task_），然后单击“添加”按钮。

  	![](./media/mobile-services-javascript-backend-android-get-started-push/mobile-quickstart-push1-android.png)

3. 从屏幕顶部向下轻扫，打开设备的通知抽屉以查看通知。


你已成功完成本教程。

##  故障排除

###  验证 Android SDK 版本

[AZURE.INCLUDE [验证 SDK](../includes/mobile-services-verify-android-sdk-version.md)]


##  旧代码版本

如果你要查看本教程的 Eclipse 版本，请转到[推送通知入门 (Eclipse)]。




##  <a name="next-steps"></a>后续步骤

<!---This tutorial demonstrated the basics of enabling an Android app to use Mobile Services and Notification Hubs to send push notifications. Next, consider completing the next tutorial, [Send push notifications to authenticated users], which shows how to use tags to send push notifications from a Mobile Service to only an authenticated user.

+ [Send push notifications to authenticated users]
	<br/>Learn how to use tags to send push notifications from a Mobile Service to only an authenticated user.

+ [Send broadcast notifications to subscribers]
	<br/>Learn how users can register and receive push notifications for categories they're interested in.

+ [Send template-based notifications to subscribers]
	<br/>Learn how to use templates to send push notifications from a Mobile Service, without having to craft platform-specific payloads in your back-end.
-->

通过以下主题了解有关移动服务和通知中心的详细信息：

* [数据处理入门]
  <br/>了解有关使用移动服务存储和查询数据的详细信息。

* [身份验证入门 ]
  <br/>了解如何通过移动服务对使用不同帐户类型的应用程序用户进行身份验证。

* [什么是通知中心？]
  <br/>了解有关通知中心跨所有主要的客户端平台向你的应用程序交付通知的详细信息。

* [调试通知中心应用程序]
  </br>获取有关对通知中心解决方案进行故障排除和调试的指导。

* [如何使用适用于移动服务的 Android 客户端库 ]
  <br/>了解有关如何将移动服务与 Android 一起使用的详细信息。

* [移动服务服务器脚本参考]
  <br/>了解有关如何在移动服务中实施业务逻辑的详细信息。


<!-- Anchors. -->
[Register your app for push notifications and configure Mobile Services]: #register
[Update the generated push notification code]: #update-scripts
[Insert data to receive notifications]: #test
[Next Steps]: #next-steps

<!-- Images. -->

[13]: ./media/mobile-services-windows-store-javascript-get-started-push/mobile-quickstart-push1.png
[14]: ./media/mobile-services-windows-store-javascript-get-started-push/mobile-quickstart-push2.png


<!-- URLs. -->
[推送通知入门 (Eclipse)]: /documentation/articles/mobile-services-javascript-backend-android-get-started-push-EC
[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Get started with Mobile Services]: /documentation/articles/mobile-services-android-get-started
[数据处理入门]: /documentation/articles/mobile-services-android-get-started-data
[身份验证入门 ]: /documentation/articles/mobile-services-android-get-started-users
[Get started with push notifications]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-js
[Push notifications to app users]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-get-started-push
[Authorize users with scripts]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-authorize-users-in-scripts
[JavaScript and HTML]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-js
[设置 Google Play Services SDK]: http://go.microsoft.com/fwlink/?LinkId=389801
[Windows Azure Management Portal]: https://manage.windowsazure.cn/
[如何使用适用于移动服务的 Android 客户端库 ]: /documentation/articles/mobile-services-android-how-to-use-client-library
[gcm object]: http://go.microsoft.com/fwlink/p/?LinkId=282645

[移动服务服务器脚本参考]: /documentation/articles/mobile-services-how-to-use-server-scripts
[调试通知中心应用程序]: http://go.microsoft.com/fwlink/p/?linkid=386630
[Send push notifications to authenticated users]: /documentation/articles/mobile-services-javascript-backend-android-push-notifications-app-users

[什么是通知中心？]: /documentation/articles/notification-hubs-overview
[Send broadcast notifications to subscribers]: /documentation/articles/notification-hubs-android-send-breaking-news
[Send template-based notifications to subscribers]: /documentation/articles/notification-hubs-android-send-localized-breaking-news

<!---HONumber=74-->