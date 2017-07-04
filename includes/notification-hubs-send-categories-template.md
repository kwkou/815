
本部分说明如何从 .NET 控制台应用以标记模板通知的形式发送突发新闻。

如果你使用的是 Mobile Apps，请参阅 [Add push notifications for Mobile Apps]（为 Mobile Apps 添加推送通知）教程，然后选择顶层的平台。

如果你使用的是 Java 或 PHP，请参阅[如何从 Java/PHP 使用通知中心]。你可使用[通知中心 REST 接口]通过任何后端发送通知。

如果你在完成[通知中心入门]时创建了用于发送通知的控制台应用，则跳过步骤 1-3。

1. 在 Visual Studio 中创建新的 Visual C# 控制台应用程序：
   
       ![][13]
2. 在 Visual Studio 主菜单中，依次单击“工具”、“库程序包管理器”和“程序包管理器控制台”，然后在控制台窗口中键入以下命令并按 **Enter**：
   
        Install-Package Microsoft.Azure.NotificationHubs
   
    这将使用 [Microsoft.Azure.Notification Hubs NuGet 包]添加对 Azure 通知中心 SDK 的引用。
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
5. 在上面的代码中，将 `<hub name>` 和 `<connection string with full access>` 占位符替换为你的通知中心名称和从通知中心仪表板获取的 *DefaultFullSharedAccessSignature* 的连接字符串。
6. 在 **Main** 方法中添加以下行：
   
         SendTemplateNotificationAsync();
         Console.ReadLine();
7. 生成控制台应用。

<!-- Images. -->
[13]: ./media/notification-hubs-back-end/notification-hub-create-console-app.png

<!-- URLs. -->

[通知中心入门]: /documentation/articles/notification-hubs-windows-store-dotnet-get-started-wns-push-notification/
[通知中心 REST 接口]: http://msdn.microsoft.com/library/windowsazure/dn223264.aspx
[Add push notifications for Mobile Apps]: /documentation/articles/app-service-mobile-windows-store-dotnet-get-started-push/
[如何从 Java/PHP 使用通知中心]: /documentation/articles/notification-hubs-java-push-notification-tutorial/
[Microsoft.Azure.Notification Hubs NuGet 包]: http://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/

<!---HONumber=Mooncake_1205_2016-->