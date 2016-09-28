
+ **.NET 后端 (C#)**：
	1. 在 Visual Studio 中，右键单击服务器项目并单击“管理 NuGet 包”，搜索 `Microsoft.Azure.NotificationHubs`，然后单击“安装”。这将安装通知中心库，以便从后端发送通知。

	2. 在后端的 Visual Studio 项目中，依次打开“控制器”>“TodoItemController.cs”。在文件的顶部，添加以下 `using` 语句：

	        using Microsoft.Azure.Mobile.Server.Config;
	        using Microsoft.Azure.NotificationHubs;


	3. 将 `PostTodoItem`方法替换为以下代码：
      
	        public async Task<IHttpActionResult> PostTodoItem(TodoItem item)
	        {
	            TodoItem current = await InsertAsync(item);
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
	
	            // iOS payload
	            var appleNotificationPayload = "{\"aps\":{\"alert\":\"" + item.Text + "\"}}";
	
	            try
	            {
	                // Send the push notification and log the results.
	                var result = await hub.SendAppleNativeNotificationAsync(appleNotificationPayload);
	
	                // Write the success result to the logs.
	                config.Services.GetTraceWriter().Info(result.State.ToString());
	            }
	            catch (System.Exception ex)
	            {
	                // Write the failure result to the logs.
	                config.Services.GetTraceWriter()
	                    .Error(ex.Message, null, "Push.SendAsync Error");
	            }
	            return CreatedAtRoute("Tables", new { id = current.Id }, current);
	        }

	4. 重新发布服务器项目。

+ **Node.js 后端**：
   
	1. 如果尚未执行此操作，请[下载快速启动项目](/documentation/articles/app-service-mobile-node-backend-how-to-use-server-sdk/#download-quickstart)或使用 [Azure 门户中的在线编辑器](/documentation/articles/app-service-mobile-node-backend-how-to-use-server-sdk/#online-editor)。
	
	2. 用以下代码替换 todoitem.js 表脚本：


	        var azureMobileApps = require('azure-mobile-apps'),
	            promises = require('azure-mobile-apps/src/utilities/promises'),
	            logger = require('azure-mobile-apps/src/logger');
	        
	        var table = azureMobileApps.table();
	        
	        // When adding record, send a push notification via APNS
	        // For this to work, you must have an APNS Hub already configured
	        table.insert(function (context) {
	            // For details of the Notification Hubs JavaScript SDK, 
	            // see http://aka.ms/nodejshubs
	            logger.info('Running TodoItem.insert');
	            
				// Create a payload that contains the new item Text.
	            var payload = "{\"aps\":{\"alert\":\"" + context.item.text + "\"}}";
	            
	            // Execute the insert.  The insert returns the results as a Promise,
	            // Do the push as a post-execute action within the promise flow.
	            return context.execute()
	                .then(function (results) {
	                    // Only do the push if configured
	                    if (context.push) {
	                        context.push.apns.send(null, payload, function (error) {
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

	2. 编辑本地计算机上的文件时，请重新发布服务器项目。

<!---HONumber=Mooncake_0919_2016-->