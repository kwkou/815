<properties pageTitle="Get started with push notification hubs using .NET runtime mobile services" metaKeywords="" description="Learn how to use Windows Azure .Net runtime mobile services and Notification Hubs to send push notifications to your Windows phone app." metaCanonical="" services="mobile" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services" authors="wesmc"  solutions="" writer="wesmc" manager="" editor=""  />

# 推送通知移动服务入门

<div class="dev-center-tutorial-selector sublanding">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-push" title="Windows Store C#">Windows 应用商店 C\#</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-push" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-push" title="Windows Phone" class="current">Windows Phone</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started-push/" title="Android">Android</a>
</div>
<div class="dev-center-tutorial-subselector"><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-push" title=".NET backend" class="current">.NET 后端</a> | 	<a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push/"  title="JavaScript backend">JavaScript 后端</a></div>

本主题说明如何结合使用 Azure 移动服务和 .NET 后端向 Windows Phone Silverlight 8 应用程序发送推送通知。在本教程中，你将要使用 Windows Azure 通知中心为快速入门项目启用推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会使用通知中心发送一条推送通知。创建的通知中心可在移动服务中任意使用，可独立于移动服务进行管理，并可供其他应用程序和服务使用。

> [WACOM.NOTE] 移动服务与通知中心的集成功能当前以预览版提供。

本教程将指导你完成启用推送通知的以下基本步骤：

1.  [更新应用程序以注册通知][]
2.  [更新服务器以发送推送通知][]
3.  [插入数据以接收推送通知][]

本教程基于移动服务快速入门。在开始学习本教程之前，必须先完成[移动服务入门][]或[数据处理入门][]，以将项目连接到移动服务。

> [WACOM.NOTE] 若要向 Windows Phone 应用商店应用程序发送推送通知，请遵照本教程的 [Windows 应用商店应用程序][]版本。

<a id="update-app"></a>
## 更新应用程序以注册通知

只有在你注册通知通道后，你的应用程序才能接收推送通知。

1.  在 Visual Studio 中，打开文件 App.xaml.cs 并添加以下 `using` 语句：

        using Microsoft.Phone.Notification;

2.  将以下 `AcquirePushChannel` 方法添加到 `App` 类：

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
        message += key + " :" + args.Collection[key] + ", ";
                    }
        Deployment.Current.Dispatcher.BeginInvoke(() =>
                    {
        MessageBox.Show(message);
                    });
            });
        }

    此代码将检索应用程序的通道 URI（如果存在）。如果不存在，则创建该 URI。然后，将打开该通道 URI 并为 toast 通知绑定该通道 URI。完全打开通道 URI 后，将调用 `ChannelUriUpdated` 方法的处理程序，并将通道注册到已接收的推送通知。如果注册失败，则会关闭通道，使应用程序的后续执行可以重试注册。将设置 `ShellToastNotificationReceived` 处理程序，使应用程序能够在运行时接收和处理推送通知。

3.  在 App.xaml.cs 中的 `Application_Launching` 事件处理程序中，添加对新的 `AcquirePushChannel` 方法的以下调用：

        AcquirePushChannel();

    这可以确保每次加载应用程序时都会请求注册。在应用程序中，你可能只需要定期执行此注册以确保注册是最新的。

4.  按 "F5" 键以运行应用程序。将显示包含注册密钥的弹出式对话框。

5.  在 Visual Studio 中，打开 Package.appxmanifest 文件，并确保“应用程序 UI”选项卡上的“支持 Toast 通知”已设置为“是” 。

    ![][]

    这可以确保你的应用程序能够引发 toast 通知。

<a id="update-server"></a>
## 更新服务器以发送推送通知

[WACOM.INCLUDE [mobile-services-dotnet-backend-update-server-push][]]

<ol start="2">
<li><p>登录到 [Windows Azure 管理门户][]，单击“移动服务”，然后单击你的应用程序 。</p></li>

<li><p>单击“推送”选项卡，选中“启用未经身份验证的推送通知”，然后单击“保存” 。</p>

</li>
</ol>

![][1]

> [WACOM.NOTE] 本教程使用处于未经身份验证模式的 MPNS。在此模式下，MPNS 将限制可发送到某个设备通道的通知数。若要解除此限制，必须生成一个证书，然后通过单击“上载”并选择该证书来上载该证书 。有关生成证书的详细信息，请参阅[设置已经过身份验证的 Web 服务以便为 Windows Phone 发送推送通知][]。

这样，移动服务便可以连接到处于未经身份验证模式的 MPNS 以发送推送通知。

<a id="test"></a>
## 在应用程序中测试推送通知

1.  在 Visual Studio 中，按 F5 键运行应用程序。

2.  在应用程序中的文本框内输入文本“hello push”，然后单击“保存” 。

    ![][2]

    此时会将一个插入请求发送到移动服务，以存储添加的项。可以看到，应用程序收到了一条包含“hello push”字样的 toast 通知 。

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
    了解有关使用 .Net 运行时移动服务存储和查询数据的详细信息。

-   [身份验证入门][]
    了解如何通过 .Net 运行时移动服务对使用不同帐户类型的应用程序用户进行身份验证。

-   [移动服务服务器脚本参考][]
    了解有关注册和使用服务器脚本的详细信息。

-   [移动服务 .NET 操作方法概念性参考][]
    了解有关如何将移动服务与 .NET 一起使用的详细信息。

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-push "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-push "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-push "Windows Phone"
  [Android]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started-push/ "Android"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-push ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push/ "JavaScript 后端"
  [更新应用程序以注册通知]: #update-app
  [更新服务器以发送推送通知]: #update-server
  [插入数据以接收推送通知]: #test
  [移动服务入门]: /zh-cn/documentation/articles/mobile-services-windows-store-get-started
  [数据处理入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-data
  [Windows 应用商店应用程序]: mobile-services-dotnet-backend-windows-store-dotnet-get-started-push
  [0]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-push/mobile-app-enable-push-wp8.png
  [mobile-services-dotnet-backend-update-server-push]: ../includes/mobile-services-dotnet-backend-update-server-push.md
  [Windows Azure 管理门户]: %20https://manage.windowsazure.cn/
  [1]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-push/mobile-push-tab.png
  [设置已经过身份验证的 Web 服务以便为 Windows Phone 发送推送通知]: http://msdn.microsoft.com/zh-cn/library/windowsphone/develop/ff941099(v=vs.105).aspx
  [2]: ./media/mobile-services-dotnet-backend-windows-phone-get-started-push/mobile-quickstart-push3-wp8.png
  [通知中心入门]: /zh-cn/manage/services/notification-hubs/getting-started-windows-dotnet/
  [向订户发送通知]: /zh-cn/manage/services/notification-hubs/breaking-news-dotnet/
  [向用户发送通知]: /zh-cn/manage/services/notification-hubs/notify-users/
  [向用户发送跨平台通知]: /zh-cn/manage/services/notification-hubs/notify-users-xplat-mobile-services/
  [身份验证入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-users
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/?LinkId=262293
  [移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library
