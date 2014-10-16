<properties linkid="develop-notificationhubs-tutorials-get-started-windowsdotnet" urlDisplayName="Get started with notification hubs" pageTitle="Get started with Azure Notification Hubs" metaKeywords="" description="Learn how to use Azure Notification Hubs to push notifications." metaCanonical="" services="notification-hubs" documentationCenter="Mobile" title="Getting Started with Notification Hubs" authors="sethm" solutions="" manager="" editor="" />

# 通知中心入门

<div class="dev-center-tutorial-selector sublanding"><a href="/en-us/documentation/articles/notification-hubs-windows-store-dotnet-get-started/" title="Windows 应用商店 C#" class="current">Windows 应用商店 C#</a><a href="/en-us/documentation/articles/notification-hubs-windows-phone-get-started/" title="Windows Phone">Windows Phone</a><a href="/en-us/documentation/articles/notification-hubs-ios-get-started/" title="iOS">iOS</a><a href="/en-us/documentation/articles/notification-hubs-android-get-started/" title="Android">Android</a><a href="/en-us/documentation/articles/notification-hubs-kindle-get-started/" title="Kindle">Kindle</a><a href="/en-us/documentation/articles/partner-xamarin-notification-hubs-ios-get-started/" title="Xamarin.iOS">Xamarin.iOS</a><a href="/en-us/documentation/articles/partner-xamarin-notification-hubs-android-get-started/" title="Xamarin.Android">Xamarin.Android</a></div>

本主题说明如何使用 Azure 通知中心向 Windows 应用商店应用程序发送推送通知。
在本教程中，你将要创建一个使用 Windows 推送通知服务 (WNS) 接收推送通知的空白 Windows 应用商店应用程序。完成后，你将能使用通知中心将推送通知广播到运行你的应用程序的所有设备。

本教程将指导你完成启用推送通知的以下基本步骤：

1.  [为推送通知注册应用程序][为推送通知注册应用程序]
2.  [配置通知中心][配置通知中心]
3.  [将你的应用程序连接到通知中心][将你的应用程序连接到通知中心]
4.  [从后端发送通知][从后端发送通知]

本教程演示使用通知中心的简单广播方案。请确保随后学习下一教程以了解如何使用通知中心来发送到特定用户和设备组。本教程需要的内容如下：

-   Microsoft Visual Studio 2012 Express for Windows 8
-   有效的 Windows 应用商店帐户

完成本教程是学习有关 Windows 应用商店应用程序的所有其他通知中心教程的先决条件。

<div class="dev-callout"><strong>说明</strong> <p>若要完成本教程，你必须有一个有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.cn/zh-cn/pricing/free-trial/" target="_blank">Azure 免费试用</a>。</p></div>

## <a name="register"></a><span class="short-header">注册应用程序</span>向 Windows 应用商店注册应用程序

要从移动服务将推送通知发送到 Windows 应用商店应用程序，你必须将你的应用程序提交到 Windows 应用商店。然后必须将通知中心配置为与 WNS 集成。

1.  如果尚未注册应用程序，请在开发人员中心内导航到 Windows 应用商店应用程序的[“提交应用程序”页][“提交应用程序”页]，用 Microsoft 帐户登录，然后单击“应用程序名称”。

    ![][]

2.  在“应用程序名称”中为应用程序键入一个名称，单击“保留应用程序名称”，然后单击“保存”。

    ![][1]

    此操作为应用程序创建一个新的 Windows 应用商店注册。

3.  在 Visual Studio 2012 Express for Windows 8 中，使用“空白应用程序”模板创建新的 Visual C# Windows 应用商店项目。

    ![][2]

4.  在解决方案资源管理器中，右键单击项目，单击“应用商店”，然后单击“将应用程序与应用商店关联...”。

    ![][3]

    将显示“将应用程序与 Windows 应用商店关联”向导。

5.  在该向导中，单击“登录”，然后用你的 Microsoft 帐户登录。

6.  单击在第 2 步中注册的应用程序，单击“下一步”，然后单击“关联”。

    ![][4]

    这会将所需的 Windows 应用商店注册信息添加到应用程序清单中。

7.  回到新应用程序的“Windows 开发人员中心”页，单击“服务”。

    ![][5]

8.  在“服务”页中，单击“Azure 移动服务”下的“Live 服务站点”。

    ![][6]

9.  单击“正在对你的服务进行身份验证”，然后记下“客户端密钥”和“程序包安全标识符(SID)”的值。

    ![][7]

    <div class="dev-callout"><b>安全说明</b>
<p>客户端密钥和程序包 SID 是重要的安全凭据。请勿将这些值告知任何人或随你的应用程序分发它们。</p>
</div>

## <a name="configure-hub"></a><span class="short-header">配置通知中心</span>配置通知中心

1.  登录到 [Azure 管理门户][Azure 管理门户]，然后单击屏幕底部的“新建”。

2.  依次单击“应用程序服务”、“Service Bus”、“通知中心”和“快速创建”。

    ![][8]

3.  键入通知中心的名称，选择所需的区域，然后单击“创建新的通知中心”。

    ![][9]

4.  单击刚刚创建的命名空间（通常为***通知中心名称*-ns**），然后单击顶部的“配置”选项卡。

    ![][10]

5.  选择顶部的“通知中心”选项卡，然后单击你刚刚创建的通知中心。

    ![][11]

6.  选择顶部的“配置”选项卡，输入你在前一部分中从 WNS 获取的“客户端机密”和“包 SID”值，然后单击“保存”。

    ![][12]

7.  选择顶部的“仪表板”选项卡，然后单击“连接信息”。记下两个连接字符串。

    ![][13]

你的通知中心现在已配置为使用 WNS，并且你有连接字符串用于注册你的应用程序和发送通知。

## <a name="connecting-app"></a><span class="short-header">连接应用程序</span>将应用程序连接到通知中心

1.  使用 [WindowsAzure.Messaging.Managed NuGet 包][WindowsAzure.Messaging.Managed NuGet 包]添加对 Windows 应用商店的 Azure 消息传送库的引用。在 Visual Studio 主菜单中，依次单击“工具”、“库程序包管理器”和“程序包管理器控制台”。然后，在控制台窗口中键入：

        Install-Package WindowsAzure.Messaging.Managed

    并按 **Enter**。

2.  打开文件 App.xaml.cs 并添加以下 `using` 语句：

        using Windows.Networking.PushNotifications;
        using Microsoft.WindowsAzure.Messaging;
        using Windows.UI.Popups;

3.  将以下代码添加到 App.xaml.cs 中的 `App` 类。确保将“中心名称”占位符替换为在门户的“通知中心”选项卡中显示的通知中心名称（如上例中的 **mynotificationhub2**）：

        private async void InitNotificationsAsync()
        {
            var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

            var hub = new NotificationHub("<hub name>", "<connection string with listen access>");              
            var result = await hub.RegisterNativeAsync(channel.Uri);

            // Displays the registration ID so you know it was successful
            if (result.RegistrationId != null)
            {
                var dialog = new MessageDialog("Registration successful: " + result.RegistrationId);
                dialog.Commands.Add(new UICommand("OK"));
                await dialog.ShowAsync();
            }

        }

    确保插入你的中心名称以及在前一部分中获取的名为 **DefaultListenSharedAccessSignature** 的连接字符串。
    此代码从 WNS 检索应用程序的 ChannelURI，然后将该 ChannelURI 注册到你的通知中心。

4.  在 App.xaml.cs 中 **OnLaunched** 事件处理程序的顶部，添加对新的 **InitNotificationsAsync** 方法的以下调用：

        InitNotificationsAsync();

    这保证每次启动应用程序时都在通知中心注册 ChannelURI。

5.  按 **F5** 键以运行应用程序。将显示包含注册密钥的弹出式对话框。

    ![][14]

## <a name="send"></a><span class="short-header">发送通知</span>从后端发送通知

你可以使用通知中心通过 [REST 接口][REST 接口]从任意后端发送通知。在本教程中，你使用 .NET 控制台应用程序和移动服务来发送通知，通过节点脚本来执行这些操作。

使用 .NET 应用程序发送通知：

1.  仍使以前的解决方案在 Visual Studio 中处于打开状态，在解决方案资源管理器中双击 **Package.appxmanifest** 以在 Visual Studio 编辑器中将其打开。

2.  向下滚动至“所有图像资产”，然后单击“徽章徽标”。在“通知”中，将“支持 Toast 通知”设置为“是”：

    ![][15]

    在“文件”菜单中，单击“全部保存”。

3.  现在创建新的 Visual C# 控制台应用程序：

    ![][16]

4.  通过使用 [WindowsAzure.ServiceBus NuGet 包][WindowsAzure.ServiceBus NuGet 包]添加对 Azure Service Bus SDK 的引用。在 Visual Studio 主菜单中，依次单击“工具”、“库程序包管理器”和“程序包管理器控制台”。然后，在控制台窗口中键入：

        Install-Package WindowsAzure.ServiceBus

    并按 **Enter**。

5.  打开文件 Program.cs 并添加以下 `using` 语句：

        using Microsoft.ServiceBus.Notifications;

6.  在`Program` 类中添加以下方法。确保将“中心名称”占位符替换为在门户的“通知中心”选项卡中显示的通知中心名称：

        private static async void SendNotificationAsync()
        {
            NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString("<connection string with full access>", "<hub name>");
            var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">Hello from a .NET App!</text></binding></visual></toast>";
            await hub.SendWindowsNativeNotificationAsync(toast);
        }

    确保插入你的中心名称以及在“配置通知中心”一节中获取的名为 **DefaultFullSharedAccessSignature** 的连接字符串。请注意这是具有“完全”访问权限（而非“侦听”访问权限）的连接字符串。

7.  然后将以下行添加到`Main` method:

         SendNotificationAsync();
         Console.ReadLine();

8.  按 **F5** 键以运行应用程序。你应接收 toast 通知。

    ![][17]

你可以在 MSDN 上的 [toast 目录][toast 目录]、[磁贴目录][磁贴目录]和[徽章概述][徽章概述]中查看所有可能的负载。

要使用移动服务发送通知，请按[移动服务入门][移动服务入门]中的说明操作，然后执行以下操作：

1.  登录到 [Azure 管理门户][Azure 管理门户]并单击你的移动服务。

2.  选择顶部的“计划程序”选项卡。

    ![][18]

3.  创建新的计划作业，插入名称，然后单击“按需”。

    ![][19]

4.  创建作业时，单击该作业名称。然后单击顶部栏上的“脚本”选项卡。

5.  在你的计划程序函数中插入以下脚本。确保将占位符替换为你的通知中心名称和你以前获取的 *DefaultFullSharedAccessSignature* 的连接字符串。完成后，单击底部栏上的“保存”。

        var azure = require('azure');
        var notificationHubService = azure.createNotificationHubService('<hub name>', '<connection string with full access>');
        notificationHubService.wns.sendToastText01(
            null,
            {
                text1: 'Hello from Mobile Services!!!'
            },
            function (error) {
                if (!error) {
                    console.warn("Notification successful");
                }
        });

6.  单击底部栏上的“运行一次”。你应接收 toast 通知。

## <a name="next-steps"> </a>后续步骤

在这个简单的示例中，你将通知广播到所有 Windows 设备。为了针对特定客户，请参考教程[使用通知中心将通知推送到用户][使用通知中心将通知推送到用户]。如果要按兴趣细分用户组，可以阅读[使用通知中心发送突发新闻][使用通知中心发送突发新闻]。若要了解有关如何使用通知中心的详细信息，请参阅[通知中心指南][通知中心指南]。

<!-- Anchors. -->  

<!-- URLs. -->

  [Windows 应用商店 C\#]: /en-us/documentation/articles/notification-hubs-windows-store-dotnet-get-started/ "Windows 应用商店 C#"
  [Windows Phone]: /en-us/documentation/articles/notification-hubs-windows-phone-get-started/ "Windows Phone"
  [iOS]: /en-us/documentation/articles/notification-hubs-ios-get-started/ "iOS"
  [Android]: /en-us/documentation/articles/notification-hubs-android-get-started/ "Android"
  [Kindle]: /en-us/documentation/articles/notification-hubs-kindle-get-started/ "Kindle"
  [Xamarin.iOS]: /en-us/documentation/articles/partner-xamarin-notification-hubs-ios-get-started/ "Xamarin.iOS"
  [Xamarin.Android]: /en-us/documentation/articles/partner-xamarin-notification-hubs-android-get-started/ "Xamarin.Android"
  [为推送通知注册应用程序]: #register
  [配置通知中心]: #configure-hub
  [将你的应用程序连接到通知中心]: #connecting-app
  [从后端发送通知]: #send
  [Azure 免费试用]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [“提交应用程序”页]: http://go.microsoft.com/fwlink/p/?LinkID=266582

  [toast 目录]: http://msdn.microsoft.com/zh-cn/library/windows/apps/hh761494.aspx
  [磁贴目录]: http://msdn.microsoft.com/zh-cn/library/windows/apps/hh761491.aspx
  [徽章概述]: http://msdn.microsoft.com/zh-cn/library/windows/apps/hh779719.aspx
  [移动服务入门]: /en-us/develop/mobile/tutorials/get-started/#create-new-service
  [18]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-scheduler1.png
  [19]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-scheduler2.png
  [使用通知中心将通知推送到用户]: /en-us/manage/services/notification-hubs/notify-users-aspnet
  [使用通知中心发送突发新闻]: /en-us/manage/services/notification-hubs/breaking-news-dotnet
  [通知中心指南]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx

<!-- Images. -->
  []: ./media/notification-hubs-windows-store-dotnet-get-started/mobile-services-submit-win8-app.png
  [1]: ./media/notification-hubs-windows-store-dotnet-get-started/mobile-services-win8-app-name.png
  [2]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-create-win8-app.png
  [3]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-associate-win8-app.png
  [4]: ./media/notification-hubs-windows-store-dotnet-get-started/mobile-services-select-app-name.png
  [5]: ./media/notification-hubs-windows-store-dotnet-get-started/mobile-services-win8-edit-app.png
  [6]: ./media/notification-hubs-windows-store-dotnet-get-started/mobile-services-win8-edit2-app.png
  [7]: ./media/notification-hubs-windows-store-dotnet-get-started/mobile-services-win8-app-push-auth.png
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [8]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-create-from-portal.png
  [9]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-create-from-portal2.png
  [10]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-select-from-portal.png
  [11]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-select-from-portal2.png
  [12]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-configure-wns.png
  [13]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-connection-strings.png
  [WindowsAzure.Messaging.Managed NuGet 包]: http://nuget.org/packages/WindowsAzure.Messaging.Managed/
  [14]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-windows-reg.png
  [REST 接口]: http://msdn.microsoft.com/zh-cn/library/azure/dn223264.aspx
  [15]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-win8-app-toast.png
  [16]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-create-console-app.png
  [WindowsAzure.ServiceBus NuGet 包]: http://nuget.org/packages/WindowsAzure.ServiceBus/
  [17]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-windows-toast.png



