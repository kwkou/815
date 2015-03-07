
接下来，需要更改注册推送通知的方式，以确保在尝试注册之前对用户进行身份验证。客户端应用更新取决于实现推送通知的方式。

###使用 Visual Studio 2013 Update 2 或更高版本中的添加推送通知向导

使用这种方法时，向导会在您的项目中生成新的 push.register.cs 文件。

>[WACOM.NOTE]目前只有 .NET 后端移动服务支持"添加推送通知向导"。

1. 在 Visual Studio 的解决方案资源管理器中，打开 app.xaml.cs 项目文件，然后在 **OnLaunched** 事件处理程序中注释掉或删除对 **UploadChannel** 方法的调用。 

2. 打开 push.register.cs 项目文件，并将 **UploadChannel** 方法替换为以下代码：

		public async static void UploadChannel()
		{
		    var channel = 
		        await Windows.Networking.PushNotifications.PushNotificationChannelManager
		        .CreatePushNotificationChannelForApplicationAsync();
		
		    try
		    {
		        // Create a native push notification registration.
		        await App.MobileService.GetPush().RegisterNativeAsync(channel.Uri);		        
		
		    }
		    catch (Exception exception)
		    {
		        HandleRegisterException(exception);
		    }
		}

	这可以确保使用包含已验证用户凭据的同一客户端实例完成注册。否则，注册将会失败并返回"未授权(401)"错误。

3. 打开 MainPage.xaml.cs 项目文件，并将 **OnNavigatedTo** 方法重写替换为以下代码：

	    protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            await AuthenticateAsync();            
            todolistPush.UploadChannel();
            RefreshTodoItems();
        }

	在此代码中，必须将生成的推送类名称 (`todolistPush`) 替换为向导生成的类名称，其格式通常为 <code><em>mobile_service</em>Push</code>。

###手动启用推送通知		

使用此方法时，需要将本教程中的注册代码直接添加到 app.xaml.cs 项目文件。

1. 在 Visual Studio 的解决方案资源管理器中，打开 app.xaml.cs 项目文件，然后在 **OnLaunched** 事件处理程序中注释掉或删除对 **InitNotificationsAsync** 的调用。 
 
2. 将 **InitNotificationsAsync** 方法的可访问性从  `private` 更改为  `public`，并添加  `static` 修饰符。 

3. 打开 MainPage.xaml.cs 项目文件，并将 **OnNavigatedTo** 方法重写替换为以下代码：

	    protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            await AuthenticateAsync();            
            App.InitNotificationsAsync();
            RefreshTodoItems();
        }

<!--HONumber=41-->
