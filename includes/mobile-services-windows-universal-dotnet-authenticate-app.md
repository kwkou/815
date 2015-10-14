
1. 打开共享项目文件 MainPage.cs 并将以下代码段添加到 MainPage 类：
	
		// Define a member variable for storing the signed-in user. 
        private MobileServiceUser user;

		// Define a method that performs the authentication process
		// using a Facebook sign-in. 
        private async System.Threading.Tasks.Task AuthenticateAsync()
        {
            while (user == null)
            {
                string message;
                try
                {
					// Change 'MobileService' to the name of your MobileServiceClient instance.
					// Sign-in using Facebook authentication.
                    user = await App.MobileService
                        .LoginAsync(MobileServiceAuthenticationProvider.Facebook);
                    message = 
                        string.Format("You are now signed in - {0}", user.UserId);
                }
                catch (InvalidOperationException)
                {
                    message = "You must log in. Login Required";
                }
                        
                var dialog = new MessageDialog(message);
                dialog.Commands.Add(new UICommand("OK"));
                await dialog.ShowAsync();
            }
        }

    使用 Facebook 登录对此用户进行身份验证。如果使用的标识提供商不是 Facebook，请将上述 **MobileServiceAuthenticationProvider** 的值更改为您的提供商的值。

3. 注释掉或删除现有 **OnNavigatedTo** 方法覆盖中对 **RefreshTodoItems** 方法的调用。

	这可以防止在对用户进行身份验证之前加载数据。

	>[AZURE.NOTE]若要成功地从 Windows Phone Store 8.1 应用进行身份验证，你必须在调用 **OnNavigated** 方法以及引发页面的 **Loaded** 事件之后调用 LoginAsync。在本教程中，这是通过将“登录”按钮添加到此应用而完成的。

4. 将以下代码段添加到 MainPage 类：

        private async void ButtonLogin_Click(object sender, RoutedEventArgs e)
        {
            // Login the user and then load data from the mobile service.
            await AuthenticateAsync();

            // Hide the login button and load items from the mobile service.
            this.ButtonLogin.Visibility = Windows.UI.Xaml.Visibility.Collapsed;
            await RefreshTodoItems();
        }
		
5. 在 Windows Store 应用项目中，打开 MainPage.xaml 项目文件，然后就在定义“保存”按钮的元素之前添加以下 **Button** 元素：

		<Button Name="ButtonLogin" Click="ButtonLogin_Click" 
                        Visibility="Visible">Sign in</Button>

6. 在 Windows Phone 应用项目中，紧靠在定义“保存”按钮的元素前面添加以下 **Button** 元素：

		<Button Grid.Row ="1" Grid.Column="1" Name="ButtonLogin" Click="ButtonLogin_Click" 
                        Visibility="Visible" Margin="10, 0, 0, 0">Sign in</Button> 

7. 打开共享的 App.xaml.cs 项目文件并添加以下 using 语句（如果还没有此语句）：

        using Microsoft.WindowsAzure.MobileServices;  
 
8. 在 App.xaml.cs 项目文件中，添加以下代码：

        protected override void OnActivated(IActivatedEventArgs args)
        {
			// Windows Phone 8.1 requires you to handle the respose from the WebAuthenticationBroker.
            #if WINDOWS_PHONE_APP
            if (args.Kind == ActivationKind.WebAuthenticationBrokerContinuation)
            {
				// Completes the sign-in process started by LoginAsync.
				// Change 'MobileService' to the name of your MobileServiceClient instance. 
                App.MobileService.LoginComplete(args as WebAuthenticationBrokerContinuationEventArgs);
            }
            #endif

            base.OnActivated(args);
        }

	如果 **OnActivated** 方法已存在，只需添加 `#if...#endif`  代码块。

9. 按 F5 键以运行 Windows 应用商店应用，单击“登录”按钮，然后使用所选的标识提供者来登录到此应用。

   	当您成功登录时，应用应该运行而不出现错误，您应该能够查询移动服务，并对数据进行更新。

10. 右键单击 Windows Phone 应用商店应用项目，单击“设置为启动项目”，然后重复上一步来验证 Windows Phone 应用商店应用是否也正常运行。

<!---HONumber=71-->