<properties 
	pageTitle="向移动服务应用添加推送通知 (Windows Phone) | Azure" 
	description="了解如何使用 Azure 移动服务和通知中心将推送通知发送到 Windows Phone 应用。" 
	services="mobile-services,notification-hubs" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="12/07/2015" 
	wacn.date="05/23/2016"/>


#  向移动服务应用程序添加推送通知

[AZURE.INCLUDE [mobile-services-selector-get-started-push](../includes/mobile-services-selector-get-started-push.md)]

## 概述

本主题说明如何使用 Azure 移动服务向 Windows Phone Silverlight 应用程序发送推送通知。在本教程中，你将要使用 Azure 通知中心为快速入门项目启用推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会使用通知中心发送一条推送通知。创建的通知中心可在移动服务中任意使用，可独立于移动服务进行管理，并可供其他应用程序和服务使用。

本教程是在 TodoList 示例应用程序的基础上制作的。在开始本教程之前，必须先完成主题[将移动服务添加到现有应用程序]以将项目连接到移动服务。如果尚未连接移动服务，“添加推送通知”向导可为你创建此连接。

>[AZURE.NOTE]若要向 Windows Phone 8.1 应用商店应用程序发送推送通知，请遵照本教程的 [Windows 应用商店应用程序](/documentation/articles/mobile-services-javascript-backend-windows-universal-dotnet-get-started-push/)版本。

## <a id="update-app"></a>更新应用程序以注册通知

只有在你注册通知通道后，你的应用程序才能接收推送通知。

1. 在 Visual Studio 中，打开文件 App.xaml.cs 并添加以下 `using` 语句：

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

	>[AZURE.NOTE]在本教程中，移动服务将向设备发送一条 toast 通知。而当你发送磁贴通知时，必须在通道上调用 **BindToShellTile** 方法。

3. 在 App.xaml.cs 中 **Application\_Launching** 事件处理程序的顶部，添加对新的 **AcquirePushChannel** 方法的以下调用：

        AcquirePushChannel();

	这可以确保每次加载页时都会请求注册。在应用程序中，你可能只需要定期执行此注册以确保注册是最新的。

5. 按 **F5** 键以运行应用。将显示包含注册密钥的弹出式对话框。
  
5. 在解决方案资源管理器中，展开“属性”，打开 WMAppManifest.xml 文件，单击“功能”选项卡并确保选中 **ID\_\_\_CAP\_\_\_PUSH\_NOTIFICATION** 功能。

   	![在 VS 中启用通知](./media/mobile-services-javascript-backend-windows-phone-get-started-push/mobile-app-enable-push-wp8.png)

   	这可以确保你的应用程序能够引发 toast 通知。

## <a id="update-scripts"></a>更新服务器脚本以发送推送通知

最后，您必须更新注册到 TodoItem 表上的插入操作的脚本，以便发送通知。

1. 单击“TodoItem”，单击“脚本”，然后选择“插入”。 

2. 将 insert 函数替换为以下代码，然后单击“保存”：

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

3. 单击“推送”选项卡，选中“启用未经身份验证的推送通知”，然后单击“保存”。

	这样，移动服务便可以连接到处于未经身份验证模式的 MPNS 以发送推送通知。

	>[AZURE.NOTE]本教程使用未经身份验证模式下的 MPNS。在此模式下，MPNS 将限制可发送到某个设备通道的通知数。若要解除此限制，必须生成一个证书，然后通过单击“上载”并选择该证书来上载该证书。有关生成证书的详细信息，请参阅[设置已经过身份验证的 Web 服务以便为 Windows Phone 发送推送通知]。

## <a id="test"></a>在应用程序中测试推送通知

1. 在 Visual Studio 中，按 F5 键运行应用程序。

    >[AZURE.NOTE]在 Windows Phone 模拟器测试时，你可能会遇到 401 错误“未授权的 RegistrationAuthorizationException”。由于 Windows Phone 模拟器时钟与主机电脑时钟的同步问题，在调用 `RegisterNativeAsync()` 期间可能会出现此错误。这可能会导致安全令牌被拒绝。若要解决此问题，只需在模拟器中手动设置时钟，然后再开始测试。

5. 在应用程序中，在文本框中输入文本“hello push”，单击“保存”，然后立即单击开始按钮或后退按钮以退出应用程序。

   	![在应用程序中输入文本](./media/mobile-services-javascript-backend-windows-phone-get-started-push/mobile-quickstart-push3-wp8.png)

  	此时会将一个插入请求发送到移动服务，以存储添加的项。可以看到，设备收到了一条包含 **hello push** 字样的 toast 通知。

	![收到的 Toast 通知](./media/mobile-services-javascript-backend-windows-phone-get-started-push/mobile-quickstart-push5-wp8.png)

   >[AZURE.NOTE]如果你仍未退出应用程序，则不会收到该通知。若要在应用程序处于活动状态时接收 toast 通知，你必须处理 [ShellToastNotificationReceived](http://msdn.microsoft.com/zh-cn/library/windowsphone/develop/microsoft.phone.notification.httpnotificationchannel.shelltoastnotificationreceived.aspx) 事件。

##  <a name="next-steps"></a>后续步骤

本教程演示了启用 Windows 应用商店应用程序以便使用移动服务和通知中心发送推送通知的基础知识。接下来，请考虑完成以下教程之一：

+ [将广播通知发送到订户](/documentation/articles/notification-hubs-windows-phone-send-breaking-news/)
	<br/>了解用户如何注册和接收他们感兴趣的类别的推送通知。

+ [将平台无关的通知发送到订户](/documentation/articles/notification-hubs-aspnet-cross-platform-notify-users/)
	<br/>了解如何使用模板从移动服务发送推送通知，且不会在后端中产生平台特定的负载。


通过以下主题了解有关移动服务和通知中心的详细信息：

* [Azure 通知中心 - 诊断指南](/documentation/articles/notification-hubs-diagnosing/)
  <br/>了解如何排查推送通知问题。

* [身份验证入门]
  <br/>了解如何通过移动服务对使用不同帐户类型的应用程序用户进行身份验证。

* [什么是通知中心？]
  <br/>了解有关通知中心跨所有主要的客户端平台向你的应用程序交付通知的详细信息。

* [移动服务 .NET 操作方法概念性参考]
  <br/>了解有关如何将移动服务与 .NET 一起使用的详细信息。

* [移动服务服务器脚本参考]
  <br/>了解有关如何在移动服务中实施业务逻辑的详细信息。

<!-- Anchors. -->

<!-- Images. -->


<!-- URLs. -->

[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[将移动服务添加到现有应用程序]: /documentation/articles/mobile-services-windows-phone-get-started-data/
[身份验证入门]: /documentation/articles/mobile-services-windows-phone-get-started-users/

[设置已经过身份验证的 Web 服务以便为 Windows Phone 发送推送通知]: http://msdn.microsoft.com/zh-cn/library/windowsphone/develop/ff941099(v=vs.105).aspx

[移动服务服务器脚本参考]: /documentation/articles/mobile-services-how-to-use-server-scripts/
[移动服务 .NET 操作方法概念性参考]: /documentation/articles/mobile-services-dotnet-how-to-use-client-library/

[什么是通知中心？]: /documentation/articles/notification-hubs-overview/

 

<!---HONumber=Mooncake_0118_2016-->