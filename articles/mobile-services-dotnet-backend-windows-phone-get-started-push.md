<properties 
	pageTitle="推送通知中心与 .NET 运行时移动服务用法入门" 
	description="了解如何使用通知中心和 Azure 移动服务将推送通知发送到 Windows Phone 应用程序。" 
	services="mobile-services,notification-hubs" 
	documentationCenter="windows" 
	authors="wesmc7777" 
	writer="wesmc" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="06/04/2015" 
	wacn.date="10/03/2015"/>

# 向移动服务应用程序添加推送通知

[AZURE.INCLUDE [mobile-services-selector-get-started-push-legacy](../includes/mobile-services-selector-get-started-push-legacy.md)]

##概述

本主题说明如何通过 Microsoft 推送通知服务 (MPNS)，结合使用 Azure 移动服务和 .NET 后端向 Windows Phone Silverlight 8 应用程序发送推送通知。在本教程中，你将要使用 Azure 通知中心为快速入门项目启用推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会使用通知中心发送一条推送通知。创建的通知中心可在移动服务中任意使用，可独立于移动服务进行管理，并可供其他应用程序和服务使用。

本教程基于移动服务 TodoList 项目。在开始本教程之前，必须先完成[将移动服务添加到现有应用程序]以将项目连接到移动服务。

>[AZURE.NOTE]本教程面向 Windows Phone 8.1 Silverlight 应用。如果你要生成 Windows Phone 8.1 应用商店应用程序，请参阅本教程的 [Windows 应用商店应用程序](/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-push)版本。有关 Windows Phone Silverlight 应用程序与 Windows Phone 应用商店应用程序的差别信息，请参阅 [Windows Phone Silverlight 8.1 应用程序]。

##更新应用程序以注册通知

只有在你注册通知通道后，你的应用程序才能接收推送通知。

1. 在 Visual Studio 中，打开文件 App.xaml.cs 并添加以下 `using` 语句：

        using Microsoft.Phone.Notification;

2. 将以下 `AcquirePushChannel` 方法添加到 `App` 类：

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
                    System.Exception exception = null;
                    try
                    {
                        await MobileService.GetPush()
                            .RegisterNativeAsync(CurrentChannel.ChannelUri.ToString());
                    }
                    catch (System.Exception ex)
                    {
                        CurrentChannel.Close();
                        exception = ex;
                    }
                    if (exception != null)
                    {
                        Deployment.Current.Dispatcher.BeginInvoke(() =>
                        {
                            MessageBox.Show(exception.Message, 
                                            "Registering for Push Notifications",
                                            MessageBoxButton.OK);
                        });
                    }
            });
            CurrentChannel.ShellToastNotificationReceived += 
                new EventHandler<NotificationEventArgs>((o, args) =>
                {
                    string message = "";
                    foreach (string key in args.Collection.Keys)
                    {
                        message += key + " : " + args.Collection[key] + ", ";
                    }
                    Deployment.Current.Dispatcher.BeginInvoke(() =>
                    {
                        MessageBox.Show(message);
                    });
            });
        }

    此代码将检索应用程序的通道 URI（如果存在）。如果不存在，则创建该 URI。然后，将打开该通道 URI 并为 toast 通知绑定该通道 URI。完全打开通道 URI 后，将调用 `ChannelUriUpdated` 方法的处理程序，并将通道注册到已接收的推送通知。如果注册失败，则会关闭通道，使应用程序的后续执行可以重试注册。将设置 `ShellToastNotificationReceived` 处理程序，使应用程序能够在运行时接收和处理推送通知。
    
3. 在 App.xaml.cs 中的 `Application_Launching` 事件处理程序中，添加对新的 `AcquirePushChannel` 方法的以下调用：

        AcquirePushChannel();

	这可以确保每次加载应用程序时都会请求注册。在应用程序中，你可能只需要定期执行此注册以确保注册是最新的。

4. 按 **F5** 键以运行应用。将显示包含注册密钥的弹出式对话框。
  
5. 在 Visual Studio 中，打开 Package.appxmanifest 文件，并确保“应用程序 UI”选项卡上的“支持 Toast 通知”已设置为“是”。

   	![][1]

   	这可以确保你的应用程序能够引发 toast 通知。

##更新服务器以发送推送通知

1. 在 Visual Studio 的解决方案资源管理器中，展开移动服务项目中的 **Controllers** 文件夹。打开 TodoItemController.cs 并使用以下代码更新 `PostTodoItem` 方法定义：  

        public async Task<IHttpActionResult> PostTodoItem(TodoItem item)
        {
            TodoItem current = await InsertAsync(item);
            MpnsPushMessage message = new MpnsPushMessage();
            message.XmlPayload = "<?xml version="1.0" encoding="utf-8"?>" +
                "<wp:Notification xmlns:wp="WPNotification">" +
                   "<wp:Toast>" +
                        "<wp:Text1>" + item.Text + "</wp:Text1>" +
                   "</wp:Toast> " +
                "</wp:Notification>";

            try
            {
                var result = await Services.Push.SendAsync(message);
                Services.Log.Info(result.State.ToString());
            }
            catch (System.Exception ex)
            {
                Services.Log.Error(ex.Message, null, "Push.SendAsync Error");
            }
            return CreatedAtRoute("Tables", new { id = current.Id }, current);
        }

    这段代码可在插入 Todo 项之后发送推送通知（包含所插入项的文本）。在发生错误的情况下，这段代码将添加一个错误日志条目，该条目可在管理门户中的移动服务的“日志”选项卡上查看。

2. 登录到 [Azure 管理门户]，单击“移动服务”，然后单击你的应用。

3. 单击“推送”选项卡，选中“启用未经身份验证的推送通知”，然后单击“保存”。

   	![移动服务推送选项卡](./media/mobile-services-dotnet-backend-windows-phone-get-started-push/mobile-push-tab.png)

	>[AZURE.NOTE]本教程使用未经身份验证模式下的 MPNS。在此模式下，MPNS 将限制可发送到某个设备通道的通知数。若要解除此限制，必须生成一个证书，然后通过单击“上载”并选择该证书来上载该证书<strong></strong>。有关生成证书的详细信息，请参阅<a href="http://msdn.microsoft.com/zh-cn/library/windowsphone/develop/ff941099(v=vs.105).aspx">设置已经过身份验证的 Web 服务以便为 Windows Phone 发送推送通知</a>。

这样，移动服务便可以连接到处于未经身份验证模式的 MPNS 以发送推送通知。

##启用推送通知以进行本地测试

[AZURE.INCLUDE [mobile-services-dotnet-backend-configure-local-push](../includes/mobile-services-dotnet-backend-configure-local-push.md)]


##在应用程序中测试推送通知

1. 在 Visual Studio 中，按 F5 键运行应用程序。

    >[AZURE.NOTE]在 Windows Phone 模拟器测试时，你可能会遇到 401 错误“未授权的 RegistrationAuthorizationException”。由于 Windows Phone 模拟器时钟与主机电脑时钟的同步问题，在调用 `RegisterNativeAsync()` 期间可能会出现此错误。这可能会导致安全令牌被拒绝。若要解决此问题，只需在模拟器中手动设置时钟，然后再开始测试。

2. 在应用程序中，在文本框中输入文本“hello push”，单击“保存”，然后立即单击开始按钮或后退按钮以退出应用程序。

   	![插入文本][2]

  	此时会将一个插入请求发送到移动服务，以存储添加的项。可以看到，设备收到了一条包含 **hello push** 字样的 toast 通知。  

	![收到的推送通知](./media/mobile-services-dotnet-backend-windows-phone-get-started-push/mobile-quickstart-push5-wp8.png)

   > [AZURE.NOTE]如果你仍未退出应用程序，则不会收到该通知。若要在应用程序处于活动状态时接收 toast 通知，你必须处理 [ShellToastNotificationReceived](http://msdn.microsoft.com/zh-cn/library/windowsphone/develop/microsoft.phone.notification.httpnotificationchannel.shelltoastnotificationreceived.aspx) 事件。

##后续步骤

本教程演示了有关如何使 Windows Phone 应用程序使用移动服务和通知中心发送推送通知的基础知识。建议你接下来完成下一篇教程[向经过身份验证的用户发送推送通知]，其中说明了如何使用标记来做到只将推送通知从移动服务发送到经过身份验证的用户。

建议通过以下移动服务和通知中心主题了解更多信息：

* [向应用程序添加身份验证]<br/>了解如何通过不同帐户类型（使用移动服务）对应用程序的用户进行身份验证。

* [什么是通知中心？] <br/>了解有关通知中心跨所有主要的客户端平台向你的应用程序交付通知的详细信息。

* [调试通知中心应用程序](http://go.microsoft.com/fwlink/p/?linkid=386630)</br>获取有关对通知中心解决方案进行故障排除和调试的指导。

* [移动服务 .NET 操作方法概念性参考]<br/>了解有关如何将移动服务与 .NET 一起使用的详细信息。



<!-- Images. -->


[1]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-push/mobile-app-enable-push-wp8.png
[2]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-push/mobile-quickstart-push3-wp8.png
[3]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-push/mobile-quickstart-push4-wp8.png


<!-- URLs. -->

[将移动服务添加到现有应用程序]: /documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-data
[Get started with authentication]: /documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-users
[向经过身份验证的用户发送推送通知]: /documentation/articles/mobile-services-dotnet-backend-windows-phone-push-notifications-app-users
[什么是通知中心？]: /documentation/articles/notification-hubs-overview
[Send broadcast notifications to subscribers]: /documentation/articles/notification-hubs-windows-phone-send-breaking-news
[Send template-based notifications to subscribers]: /documentation/articles/notification-hubs-windows-phone-send-localized-breaking-news
[移动服务 .NET 操作方法概念性参考]: /documentation/articles/mobile-services-html-how-to-use-client-library
[Windows Phone Silverlight 8.1 应用程序]: http://msdn.microsoft.com/zh-cn/library/windowsphone/develop/dn642082(v=vs.105).aspx
[Azure 管理门户]: https://manage.windowsazure.cn/

<!---HONumber=71-->