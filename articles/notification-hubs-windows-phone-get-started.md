<properties linkid="develop-notificationhubs-tutorials-get-started-windowsphone" urlDisplayName="Get Started" pageTitle="Get Started with Azure Notification Hubs" metaKeywords="" description="Learn how to use Azure Notification Hubs to push notifications." metaCanonical="" services="notification-hubs" documentationCenter="Mobile" title="Get started with Notification Hubs" authors="sethm" solutions="" manager="" editor="" />

# 通知中心入门

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started/" title="Windows 应用商店 C#">Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/notification-hubs-windows-phone-get-started/" title="Windows Phone" class="current">Windows Phone</a><a href="/zh-cn/documentation/articles/notification-hubs-ios-get-started/" title="iOS" class="current">iOS</a><a href="/zh-cn/documentation/articles/notification-hubs-android-get-started/" title="Android" class="current">Android</a><a href="/zh-cn/documentation/articles/notification-hubs-kindle-get-started/" title="Kindle" class="current">Kindle</a><a href="/zh-cn/documentation/articles/partner-xamarin-notification-hubs-ios-get-started/" title="Xamarin.iOS" class="current">Xamarin.iOS</a><a href="/zh-cn/documentation/articles/partner-xamarin-notification-hubs-android-get-started/" title="Xamarin.Android" class="current">Xamarin.Android</a></div>

本主题说明如何使用 Azure 通知中心向 Windows Phone 8 应用程序发送推送通知。
在本教程中，你将要创建一个使用 Microsoft 推送通知服务 (MPNS) 接收推送通知的空白 Windows Phone 8 应用程序。完成后，你将能使用通知中心将推送通知广播到运行你的应用程序的所有设备。

本教程将指导你完成启用推送通知的以下步骤：

1.  [创建通知中心][创建通知中心]
2.  [将你的应用程序连接到通知中心][将你的应用程序连接到通知中心]
3.  [从后端发送通知][从后端发送通知]

本教程演示使用通知中心的简单广播方案。请确保随后学习下一教程以了解如何使用通知中心来发送到特定用户和设备组。本教程需要的内容如下：

-   [Visual Studio 2012 Express for Windows Phone][Visual Studio 2012 Express for Windows Phone] 或更高版本。

完成本教程是学习有关 Windows Phone 8 应用程序的所有其他通知中心教程的先决条件。

<div class="dev-callout"><strong>说明</strong> <p>若要完成本教程，你必须有一个有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.cn/zh-cn/pricing/free-trial/" target="_blank">Azure 免费试用</a>。</p></div>

## <a name="configure-hub"></a><span class="short-header">创建通知中心</span>创建通知中心

1.  登录到 [Azure 管理门户][Azure 管理门户]，然后单击屏幕底部的“+新建”。

2.  依次单击“应用程序服务”、“Service Bus”、“通知中心”和“快速创建”。

    ![][]

3.  键入通知中心的名称，选择所需的区域，然后单击“创建新的通知中心”。

    ![][1]

4.  单击刚刚创建的命名空间（通常为***通知中心名称*-ns**），然后单击顶部的“配置”选项卡。

    ![][2]

5.  单击顶部的“通知中心”选项卡，然后单击你刚刚创建的通知中心。

    ![][3]

6.  单击底部的“连接信息”。记下两个连接字符串。

    ![][4]

7.  单击“配置”选项卡，然后单击“Windows Phone 通知设置”部分中的“启用未经身份验证的推送通知”复选框。

    ![][5]

你现在已有注册 Windows Phone 8 应用程序和发送通知所需的连接字符串。

<div class="dev-callout"><b>说明</b>
<p>本教程使用未经身份验证模式下的 MPNS。MPNS 未经身份验证的模式对你可以发送到每个通道的通知有一些限制。通知中心支持 <a href="http://msdn.microsoft.com/zh-cn/library/windowsphone/develop/ff941099(v=vs.105).aspx">MPNS 已经身份验证的模式</a>。 <!--Refer to [Notification Hubs How-To for Windows Phone 8] for more information on how to use MPNS authenticated mode.--></p>
</div>

## <a name="connecting-app"></a><span class="short-header">连接应用程序</span>将应用程序连接到通知中心

1.  在 Visual Studio 中创建一个新的 Windows Phone 8 应用程序。

    ![][6]

2.  使用 [WindowsAzure.Messaging.Managed NuGet 包][WindowsAzure.Messaging.Managed NuGet 包]添加对 Windows 应用商店的 Azure 消息传送库的引用。在 Visual Studio 菜单中，依次单击“工具”、“库程序包管理器”和“程序包管理器控制台”。然后，在控制台窗口中键入：

        Install-Package WindowsAzure.Messaging.Managed

    然后按 Enter。

3.  打开文件 App.xaml.cs 并添加以下 `using` 语句：

        using Microsoft.Phone.Notification;
        using Microsoft.WindowsAzure.Messaging;

4.  在 App.xaml.cs 中 **Application\_Launching** 方法的顶部添加以下代码：

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
    此代码从 MPNS 检索应用程序的 ChannelURI，然后将该 ChannelURI 注册到你的通知中心。它还保证每次启动应用程序时都在通知中心注册 ChannelURI。

    <div class="dev-callout"><b>说明</b>
<p>本教程将一个 toast 通知发送到设备。而当你发送磁贴通知时，必须在通道上调用 <strong>BindToShellTile</strong> 方法。若要同时支持 toast 通知和磁贴通知，请同时调用 <strong>BindToShellTile</strong> 和 <strong>BindToShellToast</strong>。 </p>
</div>

5.  在解决方案资源管理器中，展开“属性”，打开 WMAppManifest.xml 文件，单击“功能”选项卡并确保选中 **ID___CAP___PUSH_NOTIFICATION** 功能。

    ![][7]

    这样可确保你的应用程序可收到推送通知。

6.  按 F5 键以运行应用程序。

## <a name="send"></a><span class="short-header">发送通知</span>从后端发送通知

你可以使用通知中心通过 [REST 接口][REST 接口]从任意后端发送通知。在本教程中，我们将使用 .NET 控制台应用程序和移动服务来发送通知，通过节点脚本来执行这些操作。

使用 .NET 应用程序发送通知：

1.  创建新的 Visual C# 控制台应用程序：

    ![][8]

2.  通过使用 [WindowsAzure.ServiceBus NuGet 包][WindowsAzure.ServiceBus NuGet 包]添加对 Azure Service Bus SDK 的引用。在 Visual Studio 主菜单中，依次单击“工具”、“库程序包管理器”和“程序包管理器控制台”。然后，在控制台窗口中键入：

        Install-Package WindowsAzure.ServiceBus

    然后按 Enter。

3.  打开文件 Program.cs 并添加以下 `using` 语句：

        using Microsoft.ServiceBus.Notifications;

4.  在 `Program` 类中添加以下方法：

        private static async void SendNotificationAsync()
        {
            NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString("<connection string with full access>", "<hub name>");
            string toast = "<?xml version=\"1.0\" encoding=\"utf-8\"?>" +
                "<wp:Notification xmlns:wp=\"WPNotification\">" +
                   "<wp:Toast>" +
                        "<wp:Text1>Hello from a .NET App!</wp:Text1>" +
                   "</wp:Toast> " +
                "</wp:Notification>";
            await hub.SendMpnsNativeNotificationAsync(toast);
        }

    确保插入你的中心名称以及在“配置通知中心”一节中获取的名为 DefaultFullSharedAccessSignature 的连接字符串。请注意这是具有完全访问权限（而非侦听访问权限）的连接字符串。

5.  然后在 Main 方法中添加下列行：

         SendNotificationAsync();
         Console.ReadLine();

6.  按 F5 键以运行应用程序。你应接收 toast 通知。确保 Windows Phone 模拟器正在运行，并且你的应用程序已关闭。

可以在 MSDN 上的 [toast 目录][toast 目录]和[磁贴目录][磁贴目录]中找到所有可能的负载。

## <a name="next-steps"> </a>后续步骤

在这个简单的示例中，你已将通知广播到所有 Windows Phone 8 设备。为了针对特定客户，请参考教程[使用通知中心将通知推送到用户][使用通知中心将通知推送到用户]。如果要按兴趣细分用户组，可以阅读[使用通知中心发送突发新闻][使用通知中心发送突发新闻]。在[通知中心指南][通知中心指南]中了解通知中心的详细使用信息。

<!-- Anchors. -->  

  [创建通知中心]: #configure-hub
  [将你的应用程序连接到通知中心]: #connecting-app
  [从后端发送通知]: #send

<!-- URLs. -->
  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started/ "Windows 应用商店 C#"
  [Windows Phone]: /zh-cn/documentation/articles/notification-hubs-windows-phone-get-started/ "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/notification-hubs-ios-get-started/ "iOS"
  [Android]: /zh-cn/documentation/articles/notification-hubs-android-get-started/ "Android"
  [Kindle]: /zh-cn/documentation/articles/notification-hubs-kindle-get-started/ "Kindle"
  [Xamarin.iOS]: /zh-cn/documentation/articles/partner-xamarin-notification-hubs-ios-get-started/ "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/documentation/articles/partner-xamarin-notification-hubs-android-get-started/ "Xamarin.Android"
  [Visual Studio 2012 Express for Windows Phone]: https://go.microsoft.com/fwLink/p/?LinkID=268374
  [Azure 免费试用]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [WindowsAzure.ServiceBus NuGet 包]: http://nuget.org/packages/WindowsAzure.ServiceBus/
  [toast 目录]: http://msdn.microsoft.com/zh-cn/library/windowsphone/develop/jj662938(v=vs.105).aspx
  [磁贴目录]: http://msdn.microsoft.com/zh-cn/library/windowsphone/develop/hh202948(v=vs.105).aspx
  [使用通知中心将通知推送到用户]: /en-us/manage/services/notification-hubs/notify-users-aspnet
  [使用通知中心发送突发新闻]: /en-us/manage/services/notification-hubs/breaking-news-dotnet
  [通知中心指南]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx

<!-- Images. -->
  []: ./media/notification-hubs-windows-phone-get-started/notification-hub-create-from-portal.png
  [1]: ./media/notification-hubs-windows-phone-get-started/notification-hub-create-from-portal2.png
  [2]: ./media/notification-hubs-windows-phone-get-started/notification-hub-select-from-portal.png
  [3]: ./media/notification-hubs-windows-phone-get-started/notification-hub-select-from-portal2.png
  [4]: ./media/notification-hubs-windows-phone-get-started/notification-hub-connection-strings.png
  [5]: ./media/notification-hubs-windows-phone-get-started/notification-hub-pushauth.png
  [MPNS 已经身份验证的模式]: http://msdn.microsoft.com/zh-cn/library/windowsphone/develop/ff941099(v=vs.105).aspx
  [6]: ./media/notification-hubs-windows-phone-get-started/notification-hub-create-wp-app.png
  [WindowsAzure.Messaging.Managed NuGet 包]: http://nuget.org/packages/WindowsAzure.Messaging.Managed/
  [7]: ./media/notification-hubs-windows-phone-get-started/mobile-app-enable-push-wp8.png
  [REST 接口]: http://msdn.microsoft.com/zh-cn/library/azure/dn223264.aspx
  [8]: ./media/notification-hubs-windows-phone-get-started/notification-hub-create-console-app.png



