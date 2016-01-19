
本节演示了如何从 .NET 控制台应用以及其他任何应用发送通知。如果你使用的是移动服务，请参阅[推送入门](/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-push/)教程。如果你使用的是 Java 或 PHP，请参阅[如何从 Java/PHP 使用通知中心](/zh-cn/documentation/articles/notification-hubs-java-backend-how-to/)。你可使用[通知中心 REST 接口]通过任何后端发送通知。

以下代码可将通知发送到 Windows 应用商店、Windows Phone、iOS 和 Android 设备。

如果你在完成[通知中心入门][get-started]时创建了控制台应用程序，则跳过步骤 1-3。

1. 在 Visual Studio 中创建新的 Visual C# 控制台应用程序： 

   	![][13]

2. 在 Visual Studio 主菜单中，依次单击“工具”、“库程序包管理器”和“程序包管理器控制台”，然后在控制台窗口中键入以下命令并按 **Enter**：

        Install-Package Microsoft.Azure.NotificationHubs
 	
	这将使用 <a href="http://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/">Microsoft.Azure.Notification Hubs NuGet 包</a>添加对 Azure 通知中心 SDK 的引用。

3. 打开文件 Program.cs 并添加以下 `using` 语句：

        using Microsoft.Azure.NotificationHubs;

4. 在 `Program` 类中，添加以下方法，或替换此方法（如果已存在）：

        private static async void SendNotificationAsync()
        {
			// Define the notification hub.
		    NotificationHubClient hub = 
				NotificationHubClient.CreateClientFromConnectionString(
					"<connection string with full access>", "<hub name>");
		
		    // Create an array of breaking news categories.
		    var categories = new string[] { "World", "Politics", "Business", 
		        "Technology", "Science", "Sports"};
		
            foreach (var category in categories)
            {
                try
                {
                    // Define a Windows Store toast.
                    var wnsToast = "<toast><visual><binding template=\"ToastText01\">" 
                        + "<text id=\"1\">Breaking " + category + " News!" 
                        + "</text></binding></visual></toast>";         
                    await hub.SendWindowsNativeNotificationAsync(wnsToast, category);

                    // Define a Windows Phone toast.
                    var mpnsToast =
                        "<?xml version=\"1.0\" encoding=\"utf-8\"?>" +
                        "<wp:Notification xmlns:wp=\"WPNotification\">" +
                            "<wp:Toast>" +
                                "<wp:Text1>Breaking " + category + " News!</wp:Text1>" +
                            "</wp:Toast> " +
                        "</wp:Notification>";         
                    await hub.SendMpnsNativeNotificationAsync(mpnsToast, category);

                    // Define an iOS alert.
                    var alert = "{\"aps\":{\"alert\":\"Breaking " + category + " News!\"}}";
                    await hub.SendAppleNativeNotificationAsync(alert, category);

					// Define an Android notification.
                    var notification = "{\"data\":{\"msg\":\"Breaking " + category + " News!\"}}";
                    await hub.SendGcmNativeNotificationAsync(notification, category);
                }
                catch (ArgumentException)
                {
                    // An exception is raised when the notification hub hasn't been 
                    // registered for the iOS, Windows Store, or Windows Phone platform. 
                }
            }
		 }

	此代码将针对字符串数组中的所有 6 个标记将通知发送到 Windows 应用商店、Windows Phone 和 iOS 设备。使用标记是为了确保设备仅接收注册类别的通知。
	
	> [AZURE.NOTE]此后端代码支持 Windows 应用商店、Windows Phone、iOS 和 Android 客户端。发送方法将在尚未为特定客户端平台配置通知中心时返回错误响应。

6. 在上面的代码中，将 `<hub name>` 和 `<connection string with full access>` 占位符替换为你的通知中心的名称和你之前获取的 *DefaultFullSharedAccessSignature* 的连接字符串。

7. 在 **Main** 方法中添加以下行：

         SendNotificationAsync();
		 Console.ReadLine();

<!-- Anchors -->
[通过控制台应用]: #console
[通过移动服务]: #mobile-services
[运行应用并生成通知]: #test-app

<!-- Images. -->
[13]: ./media/notification-hubs-back-end/notification-hub-create-console-app.png

[15]: ./media/notification-hubs-back-end/notification-hub-scheduler1.png
[16]: ./media/notification-hubs-back-end/notification-hub-scheduler2.png

<!-- URLs. -->
[get-started]: /documentation/articles/notification-hubs-windows-store-dotnet-get-started/
[通知中心 REST 接口]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn223264.aspx

<!---HONumber=82-->