



当您发送模板通知时，您只需提供一组属性，在本例中，我们将发送一组包含当前新闻的本地化版本的属性，例如：

	{
		"News_English": "World News in English!",
    	"News_French": "World News in French!",
    	"News_Mandarin": "World News in Mandarin!"
	}


本部分演示如何使用控制台应用发送通知

包括的代码将广播到 Windows 应用商店和 iOS 设备，因为该后端可广播到支持的任何设备。


### 使用 C# 控制台应用程序发送通知 

使用以下代码修改前面创建的控制台应用中的 `SendTemplateNotificationAsync` 方法。请注意为何在本例中无需为不同的区域设置和平台发送多条通知。

        private static async void SendTemplateNotificationAsync()
        {
            // Define the notification hub.
            NotificationHubClient hub = 
				NotificationHubClient.CreateClientFromConnectionString(
					"<connection string with full access>", "<hub name>");

            // Sending the notification as a template notification. All template registrations that contain 
			// "messageParam" or "News_<local selected>" and the proper tags will receive the notifications. 
			// This includes APNS, GCM, WNS, and MPNS template registrations.
            Dictionary<string, string> templateParams = new Dictionary<string, string>();

            // Create an array of breaking news categories.
            var categories = new string[] { "World", "Politics", "Business", "Technology", "Science", "Sports"};
            var locales = new string[] { "English", "French", "Mandarin" };

            foreach (var category in categories)
            {
                templateParams["messageParam"] = "Breaking " + category + " News!";

                // Sending localized News for each tag too...
                foreach( var locale in locales)
                {
                    string key = "News_" + locale;

					// Your real localized news content would go here.
                    templateParams[key] = "Breaking " + category + " News in " + locale + "!";
                }

                await hub.SendTemplateNotificationAsync(templateParams, category);
            }
        }


请注意，此简单调用不管平台如何都会将本地化的新闻片段传递到**所有**设备，因为你的通知中心将生成正确的本机负载并将其传送到已订阅特定标记的所有设备。

### 使用移动服务发送通知

在移动服务计划程序中，可以使用以下脚本：

	var azure = require('azure');
    var notificationHubService = azure.createNotificationHubService('<hub name>', '<connection string with full access>');
    var notification = {
			"News_English": "World News in English!",
			"News_French": "World News in French!",
			"News_Mandarin", "World News in Mandarin!"
	}
	notificationHubService.send('World', notification, function(error) {
		if (!error) {
			console.warn("Notification successful");
		}
	});
	

<!---HONumber=Mooncake_0104_2016-->