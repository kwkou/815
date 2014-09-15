
本节演示如何通过两种不同的方式发送通知：

- [通过控制台应用程序]
- [通过移动服务]

这两种后端都会向 Windows 应用商店和 iOS 设备发送通知。您可使用[通知中心 REST 接口]通过任何后端发送通知。

<h3><a name="console"></a>通过使用 C# 的控制台应用程序发送通知</h3>

如果您在完成[通知中心入门][get-started]时创建了控制台应用程序，则跳过步骤 1-3。

1. 在 Visual Studio 中创建新的 Visual C# 控制台应用程序：

   	![][13]

2. 在 Visual Studio 主菜单中，依次单击“工具”、“库程序包管理器”和“程序包管理器控制台”，然后在控制台窗口中键入以下命令并按 **Enter**：

        Install-Package WindowsAzure.ServiceBus
 	
	这将通过使用 <a href="http://nuget.org/packages/WindowsAzure.ServiceBus/">WindowsAzure.ServiceBus NuGet 程序包</a>添加对 Azure Service Bus SDK 的引用。

3. 打开文件 Program.cs 并添加以下 `using` 语句：

        using Microsoft.ServiceBus.Notifications;

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
                    // Send a WNS notification using the template.            
                    await hub.SendWindowsNativeNotificationAsync(wnsToast, category);

                    // Define a Windows Phone toast.
                    var mpnsToast =
                        "<?xml version=\"1.0\" encoding=\"utf-8\"?>" +
                        "<wp:Notification xmlns:wp=\"WPNotification\">" +
                            "<wp:Toast>" +
                                "<wp:Text1>Breaking " + category + " News!</wp:Text1>" +
                            "</wp:Toast> " +
                        "</wp:Notification>";
                    // Send an MPNS notification using the template.            
                    await hub.SendMpnsNativeNotificationAsync(mpnsToast, category);

                    // Define an iOS alert.
                    var alert = "{\"aps\":{\"alert\":\"Breaking " + category + " News!\"}}";
                    // Send an APNS notification using the template.
                    await hub.SendAppleNativeNotificationAsync(alert, category);
                }
                catch (ArgumentException)
                {
                    // An exception is raised when the notification hub hasn't been 
                    // registered for the iOS, Windows Store, or Windows Phone platform. 
                }
            }
		 }

	此代码将针对字符串数组中的所有 6 个标记将通知发送到 Windows 应用商店、Windows Phone 和 iOS 设备。使用标记是为了确保设备仅接收注册类别的通知。
	
	<div class="dev-callout"><strong>注意</strong>
		<p>此后端代码支持 Windows 应用商店、Windows Phone 和 iOS 客户端。发送方法将在尚未为特定客户端平台配置通知中心时返回错误响应。</p>
	</div>

6. 在上面的代码中，将 `<hub name>` 和 `<connection string with full access>` 占位符替换为您的通知中心的名称和您之前获取的 *DefaultFullSharedAccessSignature* 的连接字符串。

7. 在 **Main** 方法中添加下列行：

         SendNotificationAsync();
		 Console.ReadLine();

您现在可继续[运行应用程序和生成通知]。

###<a name="mobile-services"></a>通过移动服务发送通知

若要使用移动服务发送通知，请执行下列操作：

0. 完成[移动服务入门]教程以创建您的移动服务。

1. 登录到 [Azure 管理门户]，单击“移动服务”，然后单击您的移动服务。

2. 单击“计划程序”选项卡，然后单击“创建”。

   	![][15]

3. 在“新建作业”中，键入名称，选择“按需”，然后单击对号以接受。

   	![][16]

4. 创建作业后，单击作业名称，然后在“脚本”选项卡中，在计划的作业函数中插入以下脚本：

	    var azure = require('azure');
	    var notificationHubService = azure.createNotificationHubService(
				'<hub name>', 
				'<connection string with full access>');

   		 // Create an array of breaking news categories.
		    var categories = ['World', 'Politics', 'Business', 'Technology', 'Science', 'Sports'];
		    for (var i = 0; i < categories.length; i++) {
		        // Send a WNS notification.
		        notificationHubService.wns.sendToastText01(categories[i], {
		            text1: 'Breaking ' + categories[i] + ' News!'
		        }, sendComplete);
		        // Send a MPNS notification.
		        notificationHubService.mpns.sendToast(categories[i], {
		            text1: 'Breaking ' + categories[i] + ' News!'
		        }, sendComplete);
		        // Send an APNS notification.
		        notificationHubService.apns.send(categories[i], {
		            alert: 'Breaking ' + categories[i] + ' News!'
		        }, sendComplete);
		    }

	此代码将针对字符串数组中的所有 6 个标记将通知发送到 Windows 应用商店、Windows Phone 和 iOS 设备。使用标记是为了确保设备仅接收注册类别的通知。

6. 在上面的代码中，将 `<hub name>`` 和 <connection string with full access>`占位符替换为您的通知中心的名称和您之前获取的 *DefaultFullSharedAccessSignature* 的连接字符串。

7. 在计划的作业函数后添加以下帮助器函数，然后单击“保存”。
	
        function sendComplete(error) {
 		   if (error) {
	            // An exception is raised when there are no devices registered for 
	            // the iOS, Windows Store, or Windows Phone platforms. Consider using template 
	            // notifications instead.
	            //console.warn("Notification failed:" + error);
	        }
	    }
	
	<div class="dev-callout"><strong>注意</strong>
		<p>此代码支持 Windows 应用商店、Windows Phone 和 iOS 客户端。发送方法将在不存在特定平台的注册时返回错误响应。为了避免此种情况，请考虑使用模板注册将一条通知发送给多个平台。有关示例，请参见<a href="/zh-cn/manage/services/notification-hubs/breaking-news-localized-dotnet/">使用通知中心广播本地化的突发新闻</a>。</p>
	</div>

您现在可继续[运行应用程序和生成通知]。

<!-- Anchors -->
[通过控制台应用程序]: #console
[通过移动服务]: #mobile-services
[运行应用程序和生成通知]: #test-app

<!-- Images. -->
[13]: ./media/notification-hubs-back-end/notification-hub-create-console-app.png

[15]: ./media/notification-hubs-back-end/notification-hub-scheduler1.png
[16]: ./media/notification-hubs-back-end/notification-hub-scheduler2.png

<!-- URLs. -->
[get-started]: /zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started/
[使用通知中心向用户发送通知]: ../notificationhubs/tutorial-notify-users-mobileservices.md
[移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started/#create-new-service
[Azure 管理门户]: https://manage.windowsazure.com/
[wns 对象]: http://go.microsoft.com/fwlink/p/?LinkId=260591
[通知中心指南]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx
[针对 Windows 应用商店的通知中心操作指南]: http://msdn.microsoft.com/zh-cn/library/jj927172.aspx
[通知中心 REST 接口]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn223264.aspx

