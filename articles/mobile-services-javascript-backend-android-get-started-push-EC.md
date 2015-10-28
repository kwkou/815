<properties 
	pageTitle="开始使用推送通知 (Android JavaScript) |移动开发人员中心" 
	description="了解如何使用 Azure 移动服务向 Android JavaScript 应用程序发送推送通知。" 
	services="mobile-services, notification-hubs" 
	documentationCenter="android" 
	authors="RickSaling" 
	writer="ricksal" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="06/03/2015" 
	wacn.date="07/25/2015"/>  

#  向移动服务应用程序添加推送通知

[AZURE.INCLUDE [mobile-services-selector-get-started-push](../includes/mobile-services-selector-get-started-push-EC.md)]

本主题演示如何使用 Azure 移动服务将推送通知发送到使用 Google Cloud Messaging (GCM)的 Android 应用程序。在本教程中，你将要使用 Azure 通知中心为快速入门项目启用推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。

本教程将指导你完成启用推送通知的以下基本步骤：

1. [启用 Google Cloud Messaging](#register)
2. [配置移动服务](#configure)
3. [向应用程序添加推送通知](#add-push)
4. [更新脚本以发送推送通知](#update-scripts)
5. [插入数据以接收通知](#test)


>[AZURE.NOTE]如果要查看已完成应用程序的源代码，请转到<a href="https://github.com/RickSaling/mobile-services-samples/tree/futures/GettingStartedWithPush/Android" target="_blank">此处</a>。

## 先决条件

[AZURE.INCLUDE [mobile-services-android-prerequisites](../includes/mobile-services-android-prerequisites-EC.md)]

## <a id="register"></a>启用 Google Cloud Messaging

[AZURE.INCLUDE [启用 GCM](../includes/mobile-services-enable-Google-cloud-messaging.md)]

## <a id="configure"></a>配置移动服务以发送推送请求

[AZURE.INCLUDE [mobile-services-android-configure-push](../includes/mobile-services-android-configure-push.md)]

## <a id="add-push"></a>向应用程序添加推送通知

### 验证 Android SDK 版本

[AZURE.INCLUDE [验证 SDK](../includes/mobile-services-verify-android-sdk-version-EC.md)]

下一步就是安装 Google Play Services。Google Cloud Messaging 对开发和测试提出了一些最低的 API 级别要求，清单中的 **minSdkVersion** 属性必须符合这些要求。

如果你要对某台较旧的设备进行测试，请查阅 [设置 Google Play Services SDK]，以确定此值可设置到的最小值，并相应地进行设置。

### 将 Google Play Services 添加到项目

[AZURE.INCLUDE [添加 Play Services](../includes/mobile-services-add-Google-play-services-EC.md)]

### 添加代码

[AZURE.INCLUDE [mobile-services-android-getting-started-with-push](../includes/mobile-services-android-getting-started-with-push-EC.md)]


## <a id="update-scripts"></a>在管理门户中更新已注册的插入脚本

1. 在管理门户中，单击“数据”选项卡，然后单击“TodoItem”表。 

   	![](./media/mobile-services-javascript-backend-android-get-started-push-EC/mobile-portal-data-tables.png)

2. 在 **todoitem** 中，单击“脚本”选项卡，然后选择“插入”。
   
  	![](./media/mobile-services-javascript-backend-android-get-started-push-EC/mobile-insert-script-push2.png)

   	将显示当 **TodoItem** 表中发生插入时所调用的函数。

3. 将 insert 函数替换为以下代码，然后单击“保存”：

		function insert(item, user, request) {
		// Define a payload for the Google Cloud Messaging toast notification.
		var payload = {
		    "data": {
		        "message": item.text 
		    }
		};		
		request.execute({
		    success: function() {
		        // If the insert succeeds, send a notification.
		        push.gcm.send(null, payload, {
		            success: function(pushResponse) {
		                console.log("Sent push:", pushResponse, payload);
		                request.respond();
		                },              
		            error: function (pushResponse) {
		                console.log("Error Sending push:", pushResponse);
		                request.respond(500, { error: pushResponse });
		                }
		            });
		        },
		    error: function(err) {
		        console.log("request.execute error", err)
		        request.respond();
		    }
		  });
		}

   	这将会注册一个新的插入脚本，插入成功后，该脚本将使用 [gcm 对象]向所有已注册的设备发送推送通知。

## <a id="test"></a>在应用程序中测试推送通知

你可以通过以下方式测试应用程序：使用 USB 电缆直接连接 Android 手机，或者在模拟器中使用虚拟设备。

### 设置模拟器以进行测试

当你在模拟器中运行此应用程序时，请确保使用支持 Google API 的 Android 虚拟设备 (AVD)。

1. 重新启动 Eclipse，在包资源管理器中，右键单击项目，单击“属性”，单击“Android”，选中“Google API”，然后单击“确定”。

	![](./media/mobile-services-javascript-backend-android-get-started-push-EC/mobile-services-import-android-properties.png)

  	这样便指定了项目要使用 Google API。

2. 从“窗口”中选择“Android 虚拟设备管理器”，选择你的设备，然后单击“编辑”。

	![](./media/mobile-services-javascript-backend-android-get-started-push-EC/mobile-services-android-virtual-device-manager.png)

3. 在“目标”中选择“Google API”，然后单击“确定”。

   	![](./media/mobile-services-javascript-backend-android-get-started-push-EC/mobile-services-android-virtual-device-manager-edit.png)

	这样便指定了 AVD 要使用 Google API。

### 运行测试

1. 在 Eclipse 中打开“运行”菜单，然后单击“运行”以启动应用程序。

2. 在应用程序中键入有意义的文本（例如 _A new Mobile Services task_），然后单击“添加”按钮。

  	![](./media/mobile-services-javascript-backend-android-get-started-push-EC/mobile-quickstart-push1-android.png)

3. 从屏幕顶部向下轻扫，打开设备的通知中心以查看通知。


你已成功完成本教程。


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

* [向应用程序添加身份验证][身份验证入门]
  <br/>了解如何通过移动服务对使用不同帐户类型的应用程序用户进行身份验证。

* [什么是通知中心？] 
  <br/>了解有关通知中心跨所有主要的客户端平台向你的应用程序交付通知的详细信息。

* [调试通知中心应用程序](http://go.microsoft.com/fwlink/p/?linkid=386630)
  </br>获取有关对通知中心解决方案进行故障排除和调试的指导。

* [如何使用适用于移动服务的 Android 客户端库]
  <br/>了解有关如何将移动服务与 Android 一起使用的详细信息。

* [移动服务服务器脚本参考]
  <br/>了解有关如何在移动服务中实施业务逻辑的详细信息。


<!-- Anchors. -->
[Register your app for push notifications and configure Mobile Services]: #register
[Update the generated push notification code]: #update-scripts
[Insert data to receive notifications]: #test
[Next Steps]:#next-steps

<!-- Images. -->
[13]: ./media/mobile-services-windows-store-javascript-get-started-push/mobile-quickstart-push1.png
[14]: ./media/mobile-services-windows-store-javascript-get-started-push/mobile-quickstart-push2.png


<!-- URLs. -->

[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[Get started with Mobile Services]: /documentation/articles/mobile-services-android-get-started
[数据处理入门]: /documentation/articles/mobile-services-android-get-started-data
[身份验证入门]: /documentation/articles/mobile-services-android-get-started-users
[推送通知入门]:/develop/mobile/tutorials/get-started-with-push-js 
[向应用程序用户推送通知]:/develop/mobile/tutorials/push-notifications-to-users-js 
[使用脚本为用户授权]:/develop/mobile/tutorials/authorize-users-in-scripts-js 
[JavaScript 和 HTML]:/develop/mobile/tutorials/get-started-with-push-js 
[设置 Google Play Services SDK]:http://go.microsoft.com/fwlink/?LinkId=389801 
[Azure 管理门户]:https://manage.windowsazure.com/ 
[如何使用适用于移动服务的 Android 客户端库]: /documentation/articles/mobile-services-android-how-to-use-client-library
[gcm 对象]:http://go.microsoft.com/fwlink/p/?LinkId=282645

[移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/?LinkId=262293

[Send push notifications to authenticated users]: /documentation/articles/mobile-services-javascript-backend-android-push-notifications-app-users
[什么是通知中心？]: /documentation/articles/notification-hubs-overview
[Send broadcast notifications to subscribers]: /documentation/articles/notification-hubs-android-send-breaking-news
[Send template-based notifications to subscribers]: /documentation/articles/notification-hubs-android-send-localized-breaking-news