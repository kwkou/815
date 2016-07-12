
本部分说明如何从 .NET 控制台应用以标记模板通知的形式发送突发新闻。

如果你使用的是移动服务，请参阅[推送入门](/documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-push/)教程。

如果你使用的是 Java 或 PHP，请参阅[如何从 Java/PHP 使用通知中心](/documentation/articles/notification-hubs-java-backend-how-to/)。你可使用[通知中心 REST 接口](http://msdn.microsoft.com/library/windowsazure/dn223264.aspx)通过任何后端发送通知。

如果你在完成[通知中心入门][get-started]时创建了用于发送通知的控制台应用，则跳过步骤 1-3。

1. 在 Visual Studio 中创建新的 Visual C# 控制台应用程序： 

   	![][13]

2. 在 Visual Studio 主菜单中，依次单击“工具”、“库程序包管理器”和“程序包管理器控制台”，然后在控制台窗口中键入以下命令并按 **Enter**：

        Install-Package Microsoft.Azure.NotificationHubs
 	
	这将使用 <a href="http://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/">Microsoft.Azure.Notification Hubs NuGet 包</a>添加对 Azure 通知中心 SDK 的引用。

3. 打开文件 Program.cs 并添加以下 `using` 语句：

        using Microsoft.Azure.NotificationHubs;

4. 在 `Program` 类中，添加以下方法，或替换此方法（如果已存在）：

        private static async void SendTemplateNotificationAsync()
        {
			// Define the notification hub.
		    NotificationHubClient hub = 
				NotificationHubClient.CreateClientFromConnectionString(
					"<connection string with full access>", "<hub name>");

            // Create an array of breaking news categories.
            var categories = new string[] { "World", "Politics", "Business", 
											"Technology", "Science", "Sports"};

            // Sending the notification as a template notification. All template registrations that contain 
			// "messageParam" and the proper tags will receive the notifications. 
			// This includes APNS, GCM, WNS, and MPNS template registrations.

            Dictionary<string, string> templateParams = new Dictionary<string, string>();

            foreach (var category in categories)
            {
                templateParams["messageParam"] = "Breaking " + category + " News!";            
                await hub.SendTemplateNotificationAsync(templateParams, category);
            }
		 }

	此代码将针对字符串数组中的所有 6 个标记发送模板通知。使用标记是为了确保设备仅接收注册类别的通知。

6. 在上面的代码中，将 `<hub name>` 和 `<connection string with full access>` 占位符替换为你的通知中心名称和从通知中心仪表板获取的 *DefaultFullSharedAccessSignature* 的连接字符串。

7. 在 **Main** 方法中添加以下行：

         SendTemplateNotificationAsync();
		 Console.ReadLine();

8. 生成控制台应用。

<!-- Anchors -->
[From a console app]: #console
[From Mobile Services]: #mobile-services
[Run the app and generate notifications]: #test-app

<!-- Images. -->
[13]: ./media/notification-hubs-back-end/notification-hub-create-console-app.png

[15]: ./media/notification-hubs-back-end/notification-hub-scheduler1.png
[16]: ./media/notification-hubs-back-end/notification-hub-scheduler2.png

<!-- URLs. -->
[get-started]: /documentation/articles/notification-hubs-windows-store-dotnet-get-started/

[wns object]: http://go.microsoft.com/fwlink/p/?LinkId=260591
[Notification Hubs Guidance]: http://msdn.microsoft.com/library/jj927170.aspx
[Notification Hubs How-To for Windows Store]: http://msdn.microsoft.com/library/jj927172.aspx
[Notification Hubs REST interface]: http://msdn.microsoft.com/library/windowsazure/dn223264.aspx

<!---HONumber=Mooncake_0104_2016-->