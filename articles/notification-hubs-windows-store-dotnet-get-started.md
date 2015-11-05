<properties
	pageTitle="Azure 通知中心入门（Windows 应用商店应用）| Windows Azure"
	description="在本教程中，你将了解如何使用 Azure 通知中心将通知推送到 Windows 应用商店或 Windows Phone 8.1（非 Silverlight）应用程序。"
	services="notification-hubs"
	documentationCenter="windows"
	authors="wesmc7777"
	manager="dwrede"
	editor="dwrede"/>

<tags
    ms.service="notification-hubs"
    ms.date="09/03/2015"
    wacn.date="11/02/2015"/>

# 通知中心入门

[AZURE.INCLUDE [notification-hubs-selector-get-started](../includes/notification-hubs-selector-get-started.md)]

##概述

本教程演示如何使用 Azure 通知中心将推送通知发送到 Windows 应用商店或 Windows Phone 8.1（非 Silverlight）应用程序。如果你要以 Windows Phone 8.1 Silverlight 为目标，请参阅 [Windows Phone](/documentation/articles/notification-hubs-windows-phone-get-started) 版本。在本教程中，你将创建一个空白 Windows 应用商店应用，它使用 Windows 推送通知服务 (WNS) 接收推送通知。完成后，你将能够使用通知中心将推送通知广播到运行你的应用的所有设备。

本教程演示使用通知中心的简单广播方案。请确保随后学习下一教程以了解如何使用通知中心来发送到特定用户和设备组。


##先决条件

本教程需要的内容如下：

+ Microsoft Visual Studio Express 2013 for Windows with Update 2<br/>若要创建通用应用项目，则必须使用这个版本的 Visual Studio。如果你只是想要创建 Windows 应用商店应用，则需要 Visual Studio 2012 Express for Windows 8。

+ 有效的 Windows 应用商店帐户

+ 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用](http://wacn-ppe.chinacloudsites.cn/pricing/1rmb-trial/)。

完成本教程是学习有关 Windows 应用商店应用程序的所有其他通知中心教程的先决条件。

##为 Windows 应用商店注册应用程序

若要将推送通知发送到 Windows 应用商店应用，你必须将你的应用关联到 Windows 应用商店。然后必须将通知中心配置为与 WNS 集成。

1. 如果尚未注册应用，请在开发人员中心内导航到 Windows 应用商店应用程序的[“提交应用”](http://go.microsoft.com/fwlink/p/?LinkID=26658)页，用 Microsoft 帐户登录，然后单击“应用名称”。

   	![][0]

2. 在“应用名称”中为应用键入一个名称，单击“保留应用名称”，然后单击“保存”。

   	![][1]

   	此操作为应用创建一个新的 Windows 应用商店注册。

3. 在 Visual Studio 中，使用“空白应用”模板来创建新的 Visual C# 应用商店应用项目。

   	![][2]

4. 在“解决方案资源管理器”中，右键单击 Windows 应用商店应用项目，单击“应用商店”，然后单击“将应用与应用商店关联...”。

   	![][3]

   	将显示“将应用与 Windows 应用商店关联”向导。

5. 在该向导中，单击“登录”，然后用你的 Microsoft 帐户登录。

6. 单击在第 2 步中注册的应用，单击“下一步”，然后单击“关联”。

   	![][4]

   	这会将所需的 Windows 应用商店注册信息添加到应用程序清单中。

7. （可选）对 Windows Phone 应用商店应用项目重复步骤 4-6。

7. 回到新应用的“Windows 开发人员中心”页，单击“服务”。

   	![][5]

8. 在“服务”页中，单击“Microsoft Azure 移动服务”下的“Live 服务站点”。

   	![][17]

9. 在“应用设置”选项卡中，记下“客户端机密”和“包安全标识符 (SID)”的值。

   	![][6]

 	> [AZURE.NOTE]**安全说明**：客户端密钥和程序包 SID 是重要的安全凭据。请勿将这些值告知任何人或随你的应用程序分发它们。

##配置通知中心

1. 登录到 [Azure 管理门户]，然后单击屏幕底部的“新建”。

2. 依次单击“应用程序服务”、“服务总线”、“通知中心”、“快速创建”。

   	![][7]

3. 键入通知中心的名称，选择所需的区域，然后单击“创建新的通知中心”。

   	![][8]

4. 单击刚刚创建的命名空间（通常为***通知中心名称*-ns**），然后单击顶部的“配置”选项卡。

   	![][9]

5. 选择顶部的“通知中心”选项卡，然后单击你刚刚创建的通知中心。

   	![][10]

6. 选择顶部的“配置”选项卡，输入你在前一部分中从 WNS 获取的“客户端机密”和“包 SID”值，然后单击“保存”。

   	![][11]

7. 选择顶部的“仪表板”选项卡，然后单击页面底部的“连接信息”。记下两个连接字符串。

   	![][12]

你的通知中心现在已配置为使用 WNS，并且你有连接字符串用于注册你的应用程序和发送通知。

##将你的应用程序连接到通知中心

1. 在 Visual Studio 中，右键单击该解决方案，然后单击“管理 NuGet 包”。

	此时将显示“管理 NuGet 包”对话框。

2. 搜索 `WindowsAzure.Messaging.Managed` 并单击“安装”、选择解决方案中的所有项目，然后接受使用条款。

	![][20]

	随后将使用 <a href="http://nuget.org/packages/WindowsAzure.Messaging.Managed/">WindowsAzure.Messaging.Managed NuGet 包</a>在所有项目中下载、安装并添加对 Windows 的 Azure 消息传送库的引用。

3. 打开 App.xaml.cs 项目文件并添加以下 `using` 语句。在通用项目中，此文件位于 `<project_name>.Shared` 文件夹中。

        using Windows.Networking.PushNotifications;
        using Microsoft.WindowsAzure.Messaging;
		using Windows.UI.Popups;

	

4. 同样在 App.xaml.cs 中，将以下 **InitNotificationsAsync** 方法定义添加到 **App** 类：

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

    此代码从 WNS 检索应用程序的 ChannelURI，然后将该 ChannelURI 注册到您的通知中心。

    >[AZURE.NOTE]确保将“中心名称”占位符替换为在门户的“通知中心”选项卡中显示的通知中心名称（如上例中的 **mynotificationhub2**）。另请使用在上一部分中获取的 **DefaultListenSharedAccessSignature** 连接字符串替换连接字符串占位符。

5. 在 App.xaml.cs 中 **OnLaunched** 事件处理程序的上方，添加对新 **InitNotificationsAsync** 方法的以下调用：

        InitNotificationsAsync();

    这保证每次启动应用程序时都在通知中心注册 ChannelURI。

6. 在“解决方案资源管理器”中，双击 Windows 应用商店应用的 **Package.appxmanifest**，然后将“通知”中的“支持 Toast”设置为“是”：

   	![][18]

   	在“文件”菜单中，单击“全部保存”。

7. （可选）对 Windows Phone 应用商店应用项目重复前一步骤。

8. 按 **F5** 键以运行应用。将显示包含注册密钥的弹出式对话框。

   	![][19]

9. （可选）重复上一个步骤以运行另一个项目。

你的应用现在已能够接收 toast 通知。

##从后端发送通知

你可以使用通知中心通过 <a href="http://msdn.microsoft.com/library/windowsazure/dn223264.aspx">REST 接口</a>从任意后端发送通知。在本教程中，你将使用 .NET 控制台应用程序来发送通知。有关如何从与通知中心集成的 Azure 移动服务后端中发送通知的示例，请参阅**移动服务中的推送通知入门**（[.NET 后端](/documentation/articles/mobile-services/mobile-services-dotnet-backend-windows-store-dotnet-get-started) | [JavaScript 后端](/documentation/articles/mobile-services/mobile-services-javascript-backend-windows-store-dotnet-get-started)）。有关如何使用 REST API 发送通知的示例，请参阅**如何使用 Java/PHP** ([Java](/documentation/articles/notification-hubs-java-backend-how-to) | [PHP](/documentation/articles/notification-hubs-php-backend-how-to)) 中的通知中心。

1. 右键单击解决方案，选择“添加”和“新建项目...”，单击“Visual C#”下面的“Windows”和“控制台应用程序”，然后单击“确定”。

   	![][13]

	这会将新的 Visual C# 控制台应用程序添加到解决方案。你也可以在单独的解决方案中进行此项操作。

4. 在 Visual Studio 中，依次单击“工具”、“NuGet 包管理器”和“包管理器控制台”。

	这会在 Visual Studio 中显示“包管理器控制台”。

6. 在“包管理器控制台”窗口中，将“默认项目”设置为新的控制台应用程序项目，然后在控制台窗口中执行以下命令：

        Install-Package WindowsAzure.ServiceBus

	这会添加对使用 <a href="http://nuget.org/packages/WindowsAzure.ServiceBus/">WindowsAzure.ServiceBus NuGet 包</a>的 Azure 服务总线 SDK 的引用。

5. 打开文件 Program.cs 并添加以下 `using` 语句：

        using Microsoft.ServiceBus.Notifications;

6. 在 **Program** 类中，添加以下方法：

        private static async void SendNotificationAsync()
        {
            NotificationHubClient hub = NotificationHubClient
				.CreateClientFromConnectionString("<connection string with full access>", "<hub name>");
            var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">Hello from a .NET App!</text></binding></visual></toast>";
            await hub.SendWindowsNativeNotificationAsync(toast);
        }

   	确保将“中心名称”占位符替换为在门户的“通知中心”选项卡中显示的通知中心名称。此外，使用你在“配置通知中心”部分中获取的名称为 **DefaultFullSharedAccessSignature** 的连接字符串替换连接字符串占位符。

	>[AZURE.NOTE]确保你使用的是具有**完全**访问权限的连接字符串，而不是具有**监听**访问权限的连接字符串。监听访问字符串无权发送通知。

7. 然后在 **Main** 方法中添加下列行：

         SendNotificationAsync();
		 Console.ReadLine();

8. 在 Visual Studio 中，右键单击控制台应用程序项目，然后单击“设置为启始项目”，将它设置为启始项目。然后按 **F5** 键运行应用程序。

   	![][14]

	所有已注册的设备将会收到 toast 通知。单击或点按 toast 标题可加载应用。

你可以在 MSDN 上的 [toast 目录]、[磁贴目录]和[提醒概述]主题中找到所有支持的负载。

##后续步骤

在这个简单的示例中，你将通知广播到所有 Windows 设备。为了针对特定客户，请参考教程[使用通知中心将通知推送到用户]。如果要按兴趣细分用户组，可以阅读[使用通知中心发送突发新闻]。若要了解有关如何使用通知中心的详细信息，请参阅[通知中心指南]。



<!-- Images. -->
[0]: ./media/notification-hubs-windows-store-dotnet-get-started/mobile-services-submit-win8-app.png
[1]: ./media/notification-hubs-windows-store-dotnet-get-started/mobile-services-win8-app-name.png
[2]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-create-windows-universal-app.png
[3]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-associate-win8-app.png
[4]: ./media/notification-hubs-windows-store-dotnet-get-started/mobile-services-select-app-name.png
[5]: ./media/notification-hubs-windows-store-dotnet-get-started/mobile-services-win8-edit-app.png
[6]: ./media/notification-hubs-windows-store-dotnet-get-started/mobile-services-win8-app-push-auth.png
[7]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-create-from-portal.png
[8]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-create-from-portal2.png
[9]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-select-from-portal.png
[10]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-select-from-portal2.png
[11]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-configure-wns.png
[12]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-connection-strings.png
[13]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-create-console-app.png
[14]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-windows-toast.png
[15]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-scheduler1.png
[16]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-scheduler2.png
[17]: ./media/notification-hubs-windows-store-dotnet-get-started/mobile-services-win8-edit2-app.png
[18]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-win8-app-toast.png
[19]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-windows-reg.png
[20]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-windows-universal-app-install-package.png

<!-- URLs. -->
[Azure 管理门户]: https://manage.windowsazure.com/
[通知中心指南]: http://msdn.microsoft.com/library/jj927170.aspx

[使用通知中心将通知推送到用户]: /documentation/articles/notification-hubs-aspnet-backend-windows-dotnet-notify-users
[使用通知中心发送突发新闻]: /documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news
[toast 目录]: http://msdn.microsoft.com/library/windows/apps/hh761494.aspx
[磁贴目录]: http://msdn.microsoft.com/library/windows/apps/hh761491.aspx
[提醒概述]: http://msdn.microsoft.com/library/windows/apps/hh779719.aspx
 

<!---HONumber=71-->