
接下来，需要更改注册推送通知的方式，以确保在尝试注册之前对用户进行身份验证。客户端应用更新取决于实现推送通知的方式。

### 使用 Visual Studio 2013 Update 2 或更高版本中的推送通知向导

按此方法，该向导在你的项目中生成新的 push.register.js 和 service.js 文件。

1. 在 Solution Explorer 中的 Visual Studio 内，打开 push.register.js 项目文件然后注释掉或删除对 **addEventListener** 的调用。 

2. 在 default.js 项目文件中，将现有**登录**函数替换为以下代码：
 
		// Request authentication from Mobile Services using a Facebook login.
		var login = function () {
		    return new WinJS.Promise(function (complete) {
		        client.login("facebook").done(function (results) {
		            userId = results.userId;
		            // Request a push notification channel.
		            Windows.Networking.PushNotifications
		                .PushNotificationChannelManager
		                .createPushNotificationChannelForApplicationAsync()
		                .then(function (channel) {
		                    // Register for notifications using the new channel
		                    client.push.registerNative(channel.uri);
		                });
		            refreshTodoItems();
		            var message = "You are now logged in as: " + userId;
		            var dialog = new Windows.UI.Popups.MessageDialog(message);
		            dialog.showAsync().done(complete);
		        }, function (error) {
		            userId = null;
		            var dialog = new Windows.UI.Popups
		                .MessageDialog("An error occurred during login", "Login Required");
		            dialog.showAsync().done(complete);
		        });
		    });
		}  

### 手动启用推送通知		

使用此方法时，您将本教程中的注册代码直接添加到 default.js 项目文件。

1. 在 Solution Explorer 中的 Visual Studio 内，打开 default.js 项目文件并在 **onActivated** 事件处理程序中，找到调用 **createPushNotificationChannelForApplicationAsync** 函数的代码行，代码行如下所示：

		// Request a push notification channel.
		Windows.Networking.PushNotifications
		    .PushNotificationChannelManager
		    .createPushNotificationChannelForApplicationAsync()
		    .then(function (channel) {
		        // Register for notifications using the new channel
		        client.push.registerNative(channel.uri);
		    }); 
 
2. 将此代码行移到**登录**函数，就移到对 **refreshTodoItems** 的调用之前，以便**登录**函数如下所示：
 
		// Request authentication from Mobile Services using a Facebook login.
		var login = function () {
		    return new WinJS.Promise(function (complete) {
		        client.login("facebook").done(function (results) {
		            userId = results.userId;
		            // Request a push notification channel.
		            Windows.Networking.PushNotifications
		                .PushNotificationChannelManager
		                .createPushNotificationChannelForApplicationAsync()
		                .then(function (channel) {
		                    // Register for notifications using the new channel
		                    client.push.registerNative(channel.uri);
		                });
		            refreshTodoItems();
		            var message = "You are now logged in as: " + userId;
		            var dialog = new Windows.UI.Popups.MessageDialog(message);
		            dialog.showAsync().done(complete);
		        }, function (error) {
		            userId = null;
		            var dialog = new Windows.UI.Popups
		                .MessageDialog("An error occurred during login", "Login Required");
		            dialog.showAsync().done(complete);
		        });
		    });
		}  

<!---HONumber=74-->