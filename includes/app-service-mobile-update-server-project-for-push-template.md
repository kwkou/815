在本部分中，更新现有移动应用后端项目中的代码，以便在每次添加新项目时推送通知。此功能由 Azure 通知中心的[模板](/documentation/articles/notification-hubs-templates-cross-platform-push-messages/)功能提供支持，并且启用了跨平台推送。使用模板为推送通知注册了各种客户端，单个通用推送可到达所有客户端平台。

选择以下与你的后端项目类型（[.NET 后端](#dotnet)或 [Node.js 后端](#nodejs)）匹配的一个过程。

### <a name="dotnet"></a>.NET 后端项目
1. 在 Visual Studio 中，右键单击服务器项目并单击“管理 NuGet 包”。搜索 `Microsoft.Azure.NotificationHubs`，然后单击“安装”。这将安装通知中心库，以便从后端发送通知。
2. 在服务器项目中，打开“控制器”>“TodoItemController.cs”，使用以下语句进行添加：

		using System.Collections.Generic;
		using Microsoft.Azure.NotificationHubs;
		using Microsoft.Azure.Mobile.Server.Config;
	

2. 在 **PostTodoItem** 方法中，在调用 **InsertAsync** 后添加如下代码：

        // Get the settings for the server project.
        HttpConfiguration config = this.Configuration;
        MobileAppSettingsDictionary settings = 
			this.Configuration.GetMobileAppSettingsProvider().GetMobileAppSettings();
        
        // Get the Notification Hubs credentials for the Mobile App.
        string notificationHubName = settings.NotificationHubName;
        string notificationHubConnection = settings
            .Connections[MobileAppSettingsKeys.NotificationHubConnectionString].ConnectionString;

        // Create a new Notification Hub client.
        NotificationHubClient hub = NotificationHubClient
        .CreateClientFromConnectionString(notificationHubConnection, notificationHubName);

        // Sending the message so that all template registrations that contain "messageParam"
        // will receive the notifications. This includes APNS, GCM, WNS, and MPNS template registrations.
        Dictionary<string,string> templateParams = new Dictionary<string,string>();
        templateParams["messageParam"] = item.Text + " was added to the list.";

        try
        {
            // Send the push notification and log the results.
            var result = await hub.SendTemplateNotificationAsync(templateParams);

            // Write the success result to the logs.
            config.Services.GetTraceWriter().Info(result.State.ToString());
        }
        catch (System.Exception ex)
        {
            // Write the failure result to the logs.
            config.Services.GetTraceWriter()
                .Error(ex.Message, null, "Push.SendAsync Error");
        }

	插入新项时，会发送包含 item.text 的模板通知。

4. 重新发布服务器项目。

### <a name="nodejs"></a>Node.js 后端项目

1. 如果尚未执行此操作，请[下载快速入门后端项目](/documentation/articles/app-service-mobile-node-backend-how-to-use-server-sdk/#download-quickstart)或使用 [Azure 门户预览中的在线编辑器](/documentation/articles/app-service-mobile-node-backend-how-to-use-server-sdk/#online-editor)。

2. 将 todoitem.js 文件中的现有代码替换为以下内容：

		var azureMobileApps = require('azure-mobile-apps'),
	    promises = require('azure-mobile-apps/src/utilities/promises'),
	    logger = require('azure-mobile-apps/src/logger');
	
		var table = azureMobileApps.table();
		
		table.insert(function (context) {
	    // For more information about the Notification Hubs JavaScript SDK, 
	    // see http://aka.ms/nodejshubs
	    logger.info('Running TodoItem.insert');
	    
	    // Define the template payload.
	    var payload = '{"messageParam": "' + context.item.text + '" }';  
	    
	    // Execute the insert.  The insert returns the results as a Promise,
	    // Do the push as a post-execute action within the promise flow.
	    return context.execute()
	        .then(function (results) {
	            // Only do the push if configured
	            if (context.push) {
					// Send a template notification.
	                context.push.send(null, payload, function (error) {
	                    if (error) {
	                        logger.error('Error while sending push notification: ', error);
	                    } else {
	                        logger.info('Push notification sent successfully!');
	                    }
	                });
	            }
	            // Don't forget to return the results from the context.execute()
	            return results;
	        })
	        .catch(function (error) {
	            logger.error('Error while running context.execute: ', error);
	        });
		});

		module.exports = table;  

	插入新项时，会发送包含 item.text 的模板通知。

2. 编辑本地计算机上的文件时，请重新发布服务器项目。

<!---HONumber=Mooncake_0116_2017-->