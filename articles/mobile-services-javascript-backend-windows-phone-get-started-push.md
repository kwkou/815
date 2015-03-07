<properties pageTitle="推送通知入门（Windows 应用商店）| 移动开发人员中心" metaKeywords="" description="了解如何使用 Azure 移动服务和通知中心将推送通知发送到您的 Windows 应用商店应用程序。" metaCanonical="" services="mobile" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services" authors="glenga" solutions="" manager="" editor="" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-windows-phone" ms.devlang="dotnet" ms.topic="article" ms.date="09/24/2014" ms.author="glenga" />


# 向移动服务应用程序添加推送通知

[WACOM.INCLUDE [mobile-services-selector-get-started-push-legacy](../includes/mobile-services-selector-get-started-push-legacy.md)]

本主题说明如何使用 Azure 移动服务向 Windows Phone Silverlight 应用程序发送推送通知。在本教程中，你将要使用 Azure 通知中心为快速入门项目启用推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会使用通知中心发送一条推送通知。创建的通知中心可在移动服务中任意使用，可独立于移动服务进行管理，并可供其他应用程序和服务使用。

本教程将指导你完成启用推送通知的以下基本步骤：

1. [更新应用程序以注册通知](#update-app)
2. [更新服务器脚本以发送推送通知](#update-scripts)
3. [插入数据以接收推送通知](#test)

本教程基于移动服务快速入门。在开始学习本教程之前，必须先完成[移动服务入门]或[数据处理入门]，以将项目连接到移动服务。如果尚未连接移动服务，"添加推送通知"向导将为你创建此连接。 

>[WACOM.NOTE]若要向 Windows Phone 8.1 应用商店应用程序发送推送通知，请遵照本教程的 [Windows 应用商店应用程序](/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push)版本。

##<a id="update-app"></a> 更新应用程序以注册通知

只有在你注册通知通道后，你的应用程序才能接收推送通知。

1. 在 Visual Studio 中，打开文件 App.xaml.cs 并添加以下  `using` 语句：

        using Microsoft.Phone.Notification;

3. 将以下代码添加到 App.xaml.cs：
	
        public static HttpNotificationChannel CurrentChannel { get; private set; }

        private void AcquirePushChannel()
        {
            CurrentChannel = HttpNotificationChannel.Find("MyPushChannel");

            if (CurrentChannel == null)
            {
                CurrentChannel = new HttpNotificationChannel("MyPushChannel");
                CurrentChannel.Open();
                CurrentChannel.BindToShellToast();
            }

            CurrentChannel.ChannelUriUpdated +=
                new EventHandler<NotificationChannelUriEventArgs>(async (o, args) =>
                {

                    // Register for notifications using the new channel
                    await MobileService.GetPush()
                        .RegisterNativeAsync(CurrentChannel.ChannelUri.ToString());
                });
        }

    此代码检索 ChannelURI 以查找来自 Microsoft 推送通知服务 (MPNS) （由 Windows Phone 8.x "Silverlight" 使用）的应用程序， 然后注册该 ChannelURI 以支持推送通知。

	>[WACOM.NOTE]在本教程中，移动服务将向设备发送一条 toast 通知。而当你发送磁贴通知时，必须在通道上调用 **BindToShellTile** 方法。

4. 在 App.xaml.cs 中 **Application_Launching** 事件处理程序的顶部，添加对新的 **AcquirePushChannel** 方法的以下调用：

        AcquirePushChannel();

	这可以确保每次加载页时都会请求注册。在您的应用程序中，您可能只想定期执行此注册以确保注册是最新的。 

5. 按 **F5** 键以运行应用。将显示包含注册密钥的弹出式对话框。
  
6.	在解决方案资源管理器中，展开**属性**，打开 WMAppManifest.xml 文件，单击**功能**选项卡，并确保选中 **ID___CAP___PUSH_NOTIFICATION** 功能。

   	![][1]

   	这可以确保你的应用程序能够引发 toast 通知。 

##<a id="update-scripts"></a> 更新服务器脚本以发送推送通知

最后，您必须更新注册到 TodoItem 表上的插入操作的脚本，以便发送通知。

1. 单击"TodoItem"，再单击"脚本"并选择"插入"。 

   ![][10]

2. 将 insert 函数替换为以下代码，然后单击"保存"：

		function insert(item, user, request) {
		// Define a payload for the Windows Phone toast notification.
		var payload = '<?xml version="1.0" encoding="utf-8"?>' +
		    '<wp:Notification xmlns:wp="WPNotification"><wp:Toast>' +
		    '<wp:Text1>New Item</wp:Text1><wp:Text2>' + item.text + 
		    '</wp:Text2></wp:Toast></wp:Notification>';
		
		request.execute({
		    success: function() {
		        // If the insert succeeds, send a notification.
		    	push.mpns.send(null, payload, 'toast', 22, {
		            success: function(pushResponse) {
		                console.log("Sent push:", pushResponse);
						request.respond();
		                },              
		                error: function (pushResponse) {
		                    console.log("Error Sending push:", pushResponse);
							request.respond(500, { error: pushResponse });
		                    }
		                });
		            }
		        });      
		}

	插入成功后，此插入脚本会向所有 Windows Phone 应用程序注册发送推送通知（包括插入项的文本）。

3. 单击**推送**选项卡，选中**启用未经身份验证的推送通知**，然后单击**保存**。

	>[WACOM.NOTE]完成本教程中使用较旧的移动服务的操作后，您可能会看到**推送**选项卡底部的链接上指出**启用增强推送**。单击此链接升级你的移动服务，以便与通知中心相集成。此更改无法恢复。有关如何在生产移动服务中启用增强的推送通知的详情，请参阅 <a href="http://go.microsoft.com/fwlink/p/?LinkId=391951">本指南</a>。

	![][11]

	这样，移动服务便可以连接到处于未经身份验证模式的 MPNS 以发送推送通知。

	>[WACOM.NOTE]本教程使用未经身份验证模式下的 MPNS。在此模式下，MPNS 将限制可发送到某个设备通道的通知数。若要解除此限制，必须生成一个证书，然后通过单击**上载**并选择该证书来上载该证书。有关生成证书的详细信息，请参阅[设置已经过身份验证的 Web 服务以便为 Windows Phone 发送推送通知]。

##<a id="test"></a> 在应用程序中测试推送通知

1. 在 Visual Studio 中，按 F5 键运行应用程序。

    >[WACOM.NOTE] 在 Windows Phone 仿真程序上进行测试时，可能会遇到 401 未授权的 RegistrationAuthorizationException。由于 Windows Phone 仿真程序需要将其时钟与主机 PC 同步，调用` RegisterNativeAsync()`的过程中可能出现此错误。它会导致安全令牌被拒绝。若要解决此问题，只需在测试前在模拟器中手动设置时钟即可。

5. 在应用中，在文本框中输入文本"hello push"，单击**保存**，然后立即单击开始按钮或后退按钮以离开应用程序。

   ![][4]

  	此时会将一个插入请求发送到移动服务，以存储添加的项。可以看到，应用程序收到了一条包含**hello push**字样的 toast 通知。

	![][5]

	>[WACOM.NOTE]当应用程序处于打开状态时，您不会收到通知。若要在应用程序处于活动状态时接收 toast 通知，必须处理[ShellToastNotificationReceived](http://msdn.microsoft.com/library/windowsphone/develop/microsoft.phone.notification.httpnotificationchannel.shelltoastnotificationreceived(v=vs.105).aspx） 事件。



## <a name="next-steps"> </a>后续步骤

本教程演示了启用 Windows 应用商店应用程序以便使用移动服务和通知中心发送推送通知的基础知识。接下来，请考虑完成以下教程之一：

+ [向经过身份验证的用户发送推送通知]
	<br/>了解如何使用标记将推送通知从移动服务发送到只有经过身份验证的用户。

+ [将广播通知发送到订阅用户]
	<br/>了解用户如何注册并接收其感兴趣的类别的推送通知。

<!---+ [将基于模板的通知发送到订阅用户]
	<br/>了解如何使用模板通过移动服务发送推送通知，而无需在后端处理特定于平台的负载。
-->
下面的主题介绍了有关移动服务和通知中心的详细信息：

* [数据处理入门]
  <br/>了解有关使用移动服务存储和查询数据的详细信息。

* [身份验证入门]
  <br/>了解如何通过不同帐户类型（使用移动服务）对应用程序的用户进行身份验证。

* [什么是通知中心？]
  <br/>了解有关通知中心跨所有主要的客户端平台向您的应用程序交付通知的详细信息。

* [移动服务 .NET 操作方法概念性参考]
  <br/>了解有关如何将移动服务与 .NET 一起使用的详细信息。

* [移动服务服务器脚本参考]
  <br/>了解有关如何在移动服务中实施业务逻辑的详细信息。

<!-- Anchors. -->

<!-- Images. -->
[1]: ./media/mobile-services-javascript-backend-windows-phone-get-started-push/mobile-app-enable-push-wp8.png
[2]: ./media/mobile-services-javascript-backend-windows-phone-get-started-push/mobile-quickstart-push1-wp8.png
[3]: ./media/mobile-services-javascript-backend-windows-phone-get-started-push/mobile-quickstart-push2-wp8.png
[4]: ./media/mobile-services-javascript-backend-windows-phone-get-started-push/mobile-quickstart-push3-wp8.png
[5]: ./media/mobile-services-javascript-backend-windows-phone-get-started-push/mobile-quickstart-push5-wp8.png
[10]: ./media/mobile-services-javascript-backend-windows-phone-get-started-push/mobile-insert-script-push2.png
[11]: ./media/mobile-services-javascript-backend-windows-phone-get-started-push/mobile-push-tab.png

<!-- URLs. -->
[提交应用程序页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[我的应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-data
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-users

[设置已经过身份验证的 Web 服务以便为 Windows Phone 发送推送通知]: http://msdn.microsoft.com/library/windowsphone/develop/ff941099(v=vs.105).aspx

[移动服务服务器脚本参考]: /zh-cn/documentation/articles/mobile-services-how-to-use-server-scripts/
[移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library


[向经过身份验证的用户发送推送通知]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-push-notifications-app-users/

[什么是通知中心？]: /zh-cn/documentation/articles/notification-hubs-overview/
[将广播通知发送到订阅用户]: /zh-cn/documentation/articles/notification-hubs-windows-phone-send-breaking-news/
[将基于模板的通知发送到订阅用户]: /zh-cn/documentation/articles/notification-hubs-windows-phone-send-localized-breaking-news/
