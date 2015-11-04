<properties
	pageTitle="Azure 通知中心入门（Windows Phone）| Microsoft Azure"
	description="在本教程中，你将了解如何使用 Azure 通知中心将通知推送到 Windows Phone 8 或 Windows Phone 8.1 Silverlight 应用程序。"
	services="notification-hubs"
	documentationCenter="windows"
	authors="wesmc7777"
	manager="dwrede"
	editor="dwrede"/>

<tags
	ms.service="notification-hubs"
	ms.date="09/08/2015"
	wacn.date="11/02/2015"/>
	
	
# 通知中心入门

[AZURE.INCLUDE [notification-hubs-selector-get-started](../includes/notification-hubs-selector-get-started.md)]

##概述

本主题演示如何使用 Azure 通知中心将推送通知发送到 Windows Phone 8 或 Windows Phone 8.1 Silverlight 应用程序。如果你要以 Windows Phone 8.1（非 Silverlight）为目标，请参阅 [Windows Universal](/zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started/) 版本。在本教程中，您将创建一个空白 Windows Phone 8 应用程序，它使用 Microsoft 推送通知服务 (MPNS) 接收推送通知。完成后，你将能使用通知中心将推送通知广播到运行你的应用程序的所有设备。

> [AZURE.NOTE]通知中心 Windows Phone SDK 不支持将 WNS 与 Windows Phone 8.1 Silverlight 应用配合使用。若要将 WNS（而不是 MPNS）与 Windows Phone 8.1 Silverlight 应用配合使用，请根据 [Windows Universal 入门](/documentation/articles/notification-hubs-windows-store-dotnet-get-started)中所述设置 WNS 凭据。然后，你便可以根据[通知用户](/documentation/articles/notification-hubs-aspnet-backend-windows-dotnet-notify-users)教程中所述通过后端注册，或者使用 [通知中心 REST API](http://msdn.microsoft.com/zh-cn/library/dn223264.aspx) 注册。

本教程将指导你完成启用推送通知的以下步骤：

1.  [创建通知中心][创建通知中心]
2.  [将你的应用程序连接到通知中心][将你的应用程序连接到通知中心]
3.  [从后端发送通知][从后端发送通知]

本教程演示使用通知中心的简单广播方案。请确保随后学习下一教程以了解如何使用通知中心来发送到特定用户和设备组。本教程需要的内容如下：

+ [Visual Studio 2012 Express for Windows Phone] 或更高版本。

完成本教程是学习有关 Windows Phone 8 应用程序的所有其他通知中心教程的先决条件。

<div class="dev-callout"><strong>Note</strong> <p>若要完成本教程，你必须有一个有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.cn/zh-cn/pricing/free-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fwww.windowsazure.cn%2Fzh-cn%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started%2F" target="_blank">Azure 免费试用</a>。</p></div>

##<a name="configure-hub"></a>创建通知中心

1. 登录到 [Azure 管理门户]，然后单击屏幕底部的“+新建”。

2. 依次单击“应用程序服务”、“服务总线”、“通知中心”、“快速创建”。

   	![][7]

3. 键入通知中心的名称，选择所需的区域，然后单击“创建新的通知中心”。

   	![][8]

4. 单击刚刚创建的命名空间（通常为***通知中心名称*-ns**），然后单击顶部的“配置”选项卡。

   	![][9]

5. 单击顶部的“通知中心”选项卡，然后单击你刚刚创建的通知中心。

   	![][10]

6. 单击底部的“连接信息”。记下两个连接字符串。

   	![][12]

7. 单击“配置”选项卡，然后单击“Windows Phone 通知设置”部分中的“启用未经身份验证的推送通知”复选框。

   	![][15]

你现在已有注册 Windows Phone 8 应用程序和发送通知所需的连接字符串。

<div class="dev-callout"><b>Note</b>
		<p>本教程使用未经身份验证模式下的 MPNS。MPNS 未经身份验证的模式对你可以发送到每个通道的通知有一些限制。通知中心支持 <a href="http://msdn.microsoft.com/zh-cn/library/windowsphone/develop/ff941099(v=vs.105).aspx">MPNS 身份验证模式</a>。<!--Refer to [Notification Hubs How-To for Windows Phone 8] for more information on how to use MPNS authenticated mode.--></p>
</div>

##<a name="connecting-app"></a>将应用连接到通知中心

1. 在 Visual Studio 中创建一个新的 Windows Phone 8 应用程序。

   	![][13]

	在 Visual Studio 2013 Update 2 或更高版本中，必须改为创建 Windows Phone Silverlight 应用程序。
	
	![][11]

2. 在 Visual Studio 中，右键单击该解决方案，然后单击“管理 NuGet 包”。

	此时将显示“管理 NuGet 包”对话框。

3. 搜索 `WindowsAzure.Messaging.Managed`，单击“安装”，然后接受使用条款。

	![][20]

	此时将使用 <a href="http://nuget.org/packages/WindowsAzure.Messaging.Managed/">WindowsAzure.Messaging.Managed NuGet 包</a>下载、安装并添加对 Windows 的 Azure 消息传送库的引用。

4. 打开文件 App.xaml.cs 并添加以下 `using` 语句：

        using Microsoft.Phone.Notification;
        using Microsoft.WindowsAzure.Messaging;

5. 在 App.xaml.cs 中 **Application\_Launching** 方法的顶部添加以下代码：
	
	    var channel = HttpNotificationChannel.Find("MyPushChannel");
        if (channel == null)
        {
            channel = new HttpNotificationChannel("MyPushChannel");
            channel.Open();
            channel.BindToShellToast();
        }

        channel.ChannelUriUpdated += new EventHandler<NotificationChannelUriEventArgs>(async (o, args) =>
        {
            var hub = new NotificationHub("<hub name>", "<connection string>");
            await hub.RegisterNativeAsync(args.ChannelUri.ToString());
        });

    确保插入你的中心名称以及在前一部分中获取的名为 **DefaultListenSharedAccessSignature** 的连接字符串。
    此代码从 MPNS 检索应用程序的 ChannelURI，然后将该 ChannelURI 注册到您的通知中心。它还保证每次启动应用程序时都在通知中心注册 ChannelURI。

	>[WACN.NOTE]本教程将一个 toast 通知发送到设备。而当你发送磁贴通知时，必须在通道上调用 **BindToShellTile** 方法。若要同时支持 toast 通知和磁贴通知，请同时调用 **BindToShellTile** 和 **BindToShellToast**。
    
6. 在解决方案资源管理器中，展开“属性”，打开 WMAppManifest.xml 文件，单击“功能”选项卡并确保选中 **ID\_\_\_CAP\_\_\_PUSH\_NOTIFICATION** 功能。

   	![][14]

   	这样可确保你的应用程序可收到推送通知。
	
7. 按 F5 键以运行应用。

	随后将显示注册消息。

##<a name="send"></a>从后端发送通知

你可以使用通知中心通过 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/dn223264.aspx">REST 接口</a>从任意后端发送通知。在本教程中，你将使用 .NET 控制台应用程序来发送通知。有关如何从与通知中心集成的 Azure 移动服务后端发送通知的示例，请参阅**移动服务中的推送通知入门**（[.NET 后端](/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push/) | [JavaScript 后端](/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push/)）。有关如何使用 REST API 发送通知的示例，请参阅**如何通过 Java/PHP 使用通知中心** ([Java](/zh-cn/documentation/articles/notification-hubs-java-backend-how-to/) | [PHP](/zh-cn/documentation/articles/notification-hubs-php-backend-how-to/))。

1. 右键单击解决方案，选择“添加”和“新建项目...”，单击“Visual C#”下面的“Windows”和“控制台应用程序”，然后单击“确定”。 

   	![][6]

	这会将新的 Visual C# 控制台应用程序添加到解决方案。你也可以在单独的解决方案中进行此项操作。

2. 右键单击，然后依次单击“工具”、“库包管理器”和“包管理器控制台”。

	这会显示包管理器控制台。

3. 在控制台窗口中，将“默认项目”设置为新的控制台应用程序项目，然后在控制台窗口中执行以下命令：

        Install-Package WindowsAzure.ServiceBus
    
	这将使用 <a href="http://nuget.org/packages/WindowsAzure.ServiceBus/">WindowsAzure.ServiceBus NuGet 包</a>添加对 Azure 服务总线 SDK 的引用。

4. 打开文件 Program.cs 并添加以下 `using` 语句：

        using Microsoft.ServiceBus.Notifications;

5. 在 **Program** 类中，添加以下方法：

        private static async void SendNotificationAsync()
        {
            NotificationHubClient hub = NotificationHubClient
				.CreateClientFromConnectionString("<connection string with full access>", "<hub name>");
            string toast = "<?xml version="1.0" encoding="utf-8"?>" +
                "<wp:Notification xmlns:wp="WPNotification">" +
                   "<wp:Toast>" +
                        "<wp:Text1>Hello from a .NET App!</wp:Text1>" +
                   "</wp:Toast> " +
                "</wp:Notification>";
            await hub.SendMpnsNativeNotificationAsync(toast);
        }

	确保将“中心名称”占位符替换为在门户的“通知中心”选项卡中显示的通知中心名称。此外，使用你在“配置通知中心”部分中获取的名称为 **DefaultFullSharedAccessSignature** 的连接字符串替换连接字符串占位符。

	>[WACN.NOTE]确保你使用的是具有**完全**访问权限的连接字符串，而不是具有**监听**访问权限的连接字符串。监听访问字符串无权发送通知。

6. 然后在 Main 方法中添加下列行：

         SendNotificationAsync();
		 Console.ReadLine();

7. 在 Windows Phone 模拟器正在运行且应用已关闭的情况下，将控制台应用程序项目设置为默认启动项目，然后按 F5 键运行应用。

	你将会收到 toast 通知。点按 toast 标题可加载应用。

可以在 MSDN 上的 [toast 目录]和[磁贴目录]主题中找到所有可能的负载。

## <a name="next-steps"></a>后续步骤

在这个简单的示例中，你已将通知广播到所有 Windows Phone 8 设备。若要针对特定客户，请参考教程[使用通知中心将通知推送到用户]。如果要按兴趣组划分用户，可以阅读[使用通知中心发送突发新闻]。在[通知中心指南]中了解有关如何使用通知中心的更多信息。

<!-- Anchors. -->
[创建通知中心]: #configure-hub
[将你的应用程序连接到通知中心]: #connecting-app
[从后端发送通知]: #send
[Next Steps]: #next-steps

<!-- Images. -->
[6]: ./media/notification-hubs-windows-phone-get-started/notification-hub-create-console-app.png
[7]: ./media/notification-hubs-windows-phone-get-started/notification-hub-create-from-portal.png
[8]: ./media/notification-hubs-windows-phone-get-started/notification-hub-create-from-portal2.png
[9]: ./media/notification-hubs-windows-phone-get-started/notification-hub-select-from-portal.png
[10]: ./media/notification-hubs-windows-phone-get-started/notification-hub-select-from-portal2.png
[11]: ./media/notification-hubs-windows-phone-get-started/notification-hub-create-wp-silverlight-app.png
[12]: ./media/notification-hubs-windows-phone-get-started/notification-hub-connection-strings.png

[13]: ./media/notification-hubs-windows-phone-get-started/notification-hub-create-wp-app.png
[14]: ./media/notification-hubs-windows-phone-get-started/mobile-app-enable-push-wp8.png
[15]: ./media/notification-hubs-windows-phone-get-started/notification-hub-pushauth.png
[20]: ./media/notification-hubs-windows-phone-get-started/notification-hub-windows-universal-app-install-package.png
[213]: ./media/notification-hubs-windows-phone-get-started/notification-hub-create-console-app.png





<!-- URLs. -->
  [Visual Studio 2012 Express for Windows Phone]: https://go.microsoft.com/fwLink/p/?LinkID=268374

  [通知中心指南]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx
  [Notification Hubs How-To for Windows Phone 8]: tbd!!!
  [Azure 试用]: http://www.windowsazure.cn/pricing/1rmb-trial/
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [WindowsAzure.ServiceBus NuGet 包]: http://nuget.org/packages/WindowsAzure.ServiceBus/
  [toast 目录]: http://msdn.microsoft.com/library/windowsphone/develop/jj662938(v=vs.105).aspx
  [磁贴目录]: http://msdn.microsoft.com/library/windowsphone/develop/hh202948(v=vs.105).aspx
  [使用通知中心将通知推送到用户]: /documentation/articles/notification-hubs-aspnet-backend-windows-dotnet-notify-users
  [使用通知中心发送突发新闻]: /documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news
  [通知中心指南]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx




