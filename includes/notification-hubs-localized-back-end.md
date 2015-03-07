

在您的后端应用中，您现在必须切换到发送模板通知而不是本机负载。这将简化后端代码，因为您不必为不同的平台发送多个负载。

当您发送模板通知时，您只需提供一组属性，在本例中，我们将发送一组包含当前新闻的本地化版本的属性，例如：

	{
		"News_English": "World News in English!",
    	"News_French": "World News in French!",
    	"News_Mandarin": "World News in Mandarin!"
	}


本节演示如何通过两种不同的方式发送通知：

- 使用控制台应用
- 使用移动服务脚本

包括的代码将广播到 Windows 应用商店和 iOS 设备，因为该后端可广播到支持的任何设备。



## 使用 C# 控制台应用发送通知 ##

我们将只通过发送一条模板通知来修改  *SendNotificationAsync* 方法。

	var hub = NotificationHubClient.CreateClientFromConnectionString("<connection string>", "<hub name>");
    var notification = new Dictionary<string, string>() {
							{"News_English", "World News in English!"},
                            {"News_French", "World News in French!"},
                            {"News_Mandarin", "World News in Mandarin!"}};
    await hub.SendTemplateNotificationAsync(notification, "World");

请注意，此简单调用不管平台如何都会将正确的本地化新闻传递到您的**所有**设备，因为您的通知中心将生成正确的本机负载并将其传递到已订阅特定标记的所有设备。

### 移动服务

在您的移动服务计划程序中，使用以下代码覆盖您的脚本：

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
	
请注意为何在本例中无需为不同的区域设置和平台发送多条通知。
<!--HONumber=41-->
