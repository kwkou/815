<properties 
	pageTitle="推送通知入门 (Android JavaScript) | 移动开发人员中心" 
	description="了解如何使用 Azure 移动服务向 Android JavaScript 应用程序发送推送通知。" 
	services="mobile-services, notification-hubs" 
	documentationCenter="android" 
	authors="RickSaling" 
	writer="ricksal" 
	manager="dwrede" 
	editor=""/>

<tags 
wacn.date="04/15/2015"

	ms.service="mobile-services" 
	ms.workload="mobile" 
	ms.tgt_pltfrm="mobile-android" 
	ms.devlang="java" 
	ms.topic="article" 
	ms.date="02/06/2015" 
	ms.author="ricksal"/>

# 向移动服务应用程序添加推送通知

[AZURE.INCLUDE [mobile-services-selector-get-started-push](../includes/mobile-services-selector-get-started-push-EC.md)]

本主题演示如何使用 Azure 移动服务将推送通知发送到使用 Google Cloud Messaging (GCM)的 Android 应用程序。在本教程中，你将要使用 Azure 通知中心为快速入门项目启用推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。

本教程将指导你完成启用推送通知的以下基本步骤：

1. [启用 Google Cloud Messaging](#register)
2. [配置移动服务](#configure)
3. [向应用程序添加推送通知](#add-push)
4. [更新脚本以发送推送通知](#update-scripts)
5. [插入数据以接收通知](#test)


>[AZURE.NOTE] 如果要查看已完成应用程序的源代码，请转到<a href="https://github.com/RickSaling/mobile-services-samples/tree/futures/GettingStartedWithPush/Android" target="_blank">此处</a>。

## 先决条件

[AZURE.INCLUDE [mobile-services-android-prerequisites](../includes/mobile-services-android-prerequisites-EC.md)]

## <a id="register"></a>启用 Google Cloud Messaging

>[AZURE.NOTE]若要完成此过程，你必须拥有一个包含已验证电子邮件地址的 Google 帐户。若要新建一个 Google 帐户，请转至 <a href="https://accounts.google.com/SignUp" target="_blank">accounts.google.com</a>。

[AZURE.INCLUDE [Enable GCM](../includes/mobile-services-enable-Google-cloud-messaging.md)]

接下来，你将使用此 API 密钥值，让移动服务向 GCM 进行身份验证并代表你的应用程序发送推送通知。

## <a id="configure"></a>配置移动服务以发送推送请求

1. 登录到 [Azure 管理门户]、单击"移动服务"，然后单击你的应用。

   	![](./media/mobile-services-android-get-started-push/mobile-services-selection.png)

2. 单击"推送"选项卡，输入你在执行前一过程时从 GCM 获取的 **API 密钥**值，然后单击"保存"。

	>[AZURE.NOTE]完成本教程中使用较旧的移动服务的操作后，你可能会看到"推送"选项卡底部的链接上指出"启用增强推送"。单击此链接升级你的移动服务，以便与通知中心相集成。此更改无法恢复。有关如何在生产移动服务中启用增强的推送通知的详细信息，请参阅<a href="https://msdn.microsoft.com/zh-cn/library/windowsazure/dn635173.aspx">此指南</a>。

   	![](./media/mobile-services-android-get-started-push/mobile-push-tab-android.png)

	> [AZURE.NOTE] 在门户的"推送"选项卡中为增强型推送通知设置 GCM 凭据后，这些凭据将与通知中心共享，以使用你的应用程序配置通知中心。


现在，你的移动服务和应用程序都已配置为使用 GCM 和通知中心。

## <a id="add-push"></a>向应用程序添加推送通知

### 验证 Android SDK 版本

[AZURE.INCLUDE [Verify SDK](../includes/mobile-services-verify-android-sdk-version-EC.md)]

下一步就是安装 Google Play Services。Google Cloud Messaging 对开发和测试提出了一些最低的 API 级别要求，清单中的 **minSdkVersion** 属性必须符合这些要求。 

如果你要对某台较旧的设备进行测试，请查阅[设置 Google Play Services SDK]，以确定此值可设置到的最小值，并相应地进行设置。

### 将 Google Play Services 添加到项目

[AZURE.INCLUDE [Add Play Services](../includes/mobile-services-add-Google-play-services-EC.md)]

### 添加代码

[AZURE.INCLUDE [mobile-services-android-getting-started-with-push](../includes/mobile-services-android-getting-started-with-push-EC.md)]


## <a id="update-scripts"></a>在管理门户中更新已注册的插入脚本

1. 在管理门户中，单击"数据"选项卡，然后单击 **TodoItem** 表。 

   	![](./media/mobile-services-android-get-started-push/mobile-portal-data-tables.png)

2. 在 **todoitem** 中，单击"脚本"选项卡，然后选择"插入"。
   
  	![](./media/mobile-services-android-get-started-push/mobile-insert-script-push2.png)

   	将显示当 **TodoItem** 表中发生插入时所调用的函数。

3. 将 insert 函数替换为以下代码，然后单击"保存"：

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

1. 重新启动 Eclipse，在包资源管理器中，右键单击项目，单击"属性"，单击"Android"，选中"Google API"，然后单击"确定"。

	![](./media/mobile-services-android-get-started-push/mobile-services-import-android-properties.png)

  	这样便指定了项目要使用 Google API。

2. 从"窗口"中选择"Android 虚拟设备管理器"，选择你的设备，然后单击"编辑"。

	![](./media/mobile-services-android-get-started-push/mobile-services-android-virtual-device-manager.png)

3. 在"目标"中选择"Google API"，然后单击"确定"。

   	![](./media/mobile-services-android-get-started-push/mobile-services-android-virtual-device-manager-edit.png)

	这样便指定了 AVD 要使用 Google API。

### 运行测试

1. 在 Eclipse 中打开"运行"菜单，然后单击"运行"以启动应用程序。

2. 在应用程序中键入有意义的文本（例如  _A new Mobile Services task_），然后单击"添加"按钮。

  	![](./media/mobile-services-android-get-started-push/mobile-quickstart-push1-android.png)

3. 从屏幕顶部向下轻扫，打开设备的通知中心以查看通知。


你已成功完成本教程。


## <a name="next-steps"> </a>后续步骤

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

* [身份验证入门]
  <br/>了解如何使用移动服务对不同帐户类型的应用程序用户进行身份验证。

* [什么是通知中心？]
  <br/>了解有关通知中心将通知传递到所有主要客户端平台上的应用程序的工作方式的详细信息。

* [调试通知中心应用程序](https://msdn.microsoft.com/zh-cn/library/dn530751.aspx)
  </br>获取有关对通知中心解决方案进行故障排除和调试的指导。 

* [如何使用适用于移动服务的 Android 客户端库]
  <br/>详细了解如何将移动服务与 Android 一起使用。

* [移动服务服务器脚本参考]
  <br/>了解有关如何在移动服务中实施业务逻辑的详细信息。


<!-- Anchors. -->
[注册用于推送通知的应用程序并配置移动服务]: #register
[更新生成的推送通知代码]: #update-scripts
[插入数据以接收通知]: #test
[后续步骤]:#next-steps

<!-- Images. -->
[13]: ./media/mobile-services-windows-store-javascript-get-started-push/mobile-quickstart-push1.png
[14]: ./media/mobile-services-windows-store-javascript-get-started-push/mobile-quickstart-push2.png


<!-- URLs. -->
[提交应用程序页]: https://appdev.microsoft.com/StorePortals/zh-CN/Developer/Catalog/ReleaseAnchor
[我的应用程序]: https://account.live.com/developers/applications/index
[Live SDK for Windows]: http://www.microsoft.com/zh-CN/download/details.aspx?id=42552
[移动服务入门]: /documentation/articles/mobile-services-android-get-started/
[数据处理入门]: /documentation/articles/mobile-services-android-get-started-data/
[身份验证入门]: /documentation/articles/mobile-services-android-get-started-users
[推送通知入门]: /develop/mobile/tutorials/get-started-with-push-js
[向应用程序用户推送通知]: /develop/mobile/tutorials/push-notifications-to-users-js
[使用脚本为用户授权]: /develop/mobile/tutorials/authorize-users-in-scripts-js
[JavaScript 和 HTML]: /develop/mobile/tutorials/get-started-with-push-js
[设置 Google Play Services SDK]: http://developer.android.com/google/play-services/setup.html
[Azure 管理门户]: https://manage.windowsazure.cn/
[如何使用适用于移动服务的 Android 客户端库]: /documentation/articles/mobile-services-android-how-to-use-client-library

[gcm 对象]: https://msdn.microsoft.com/zh-cn/library/dn126137.aspx

[移动服务服务器脚本参考]: /develop/mobile/how-to-guides/work-with-server-scripts/

[向经过身份验证的用户发送推送通知]: /documentation/articles/mobile-services-javascript-backend-android-push-notifications-app-users/

[什么是通知中心？]: /documentation/articles/notification-hubs-overview/
[将广播通知发送到订户]: /documentation/articles/notification-hubs-android-send-breaking-news/
[将基于模板的通知发送到订户]: /documentation/articles/notification-hubs-android-send-localized-breaking-news/

<!--HONumber=50-->