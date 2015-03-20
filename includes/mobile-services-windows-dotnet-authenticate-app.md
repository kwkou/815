
1. 打开项目文件 MainPage.xaml.cs 并添加以下 using 语句：

        using Windows.UI.Popups;

2. 将以下代码段添加到 MainPage 类：
	
        private MobileServiceUser user;
        private async System.Threading.Tasks.Task AuthenticateAsync()
        {
            while (user == null)
            {
                string message;
                try
                {
                    user = await App.MobileService
                        .LoginAsync(MobileServiceAuthenticationProvider.Facebook);
                    message = 
                        string.Format("You are now logged in - {0}", user.UserId);
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

    这样可以创建用于存储当前用户的成员变量，以及用于处理身份验证过程的方法。将使用 Facebook 登录对用户进行身份验证。如果使用的标识提供商不是 Facebook，请将上述 **MobileServiceAuthenticationProvider** 的值更改为您的提供商的值。

3. 将现有的 **OnNavigatedTo** 方法覆盖替换为以下方法，以便调用新的 **Authenticate** 方法：

        protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            await AuthenticateAsync();
            RefreshTodoItems();
        }
		
4. 按 F5 键运行应用，并使用您选择的标识提供程序登录应用。 

   	当您成功登录时，应用应该运行而不出现错误，您应该能够查询移动服务，并对数据进行更新。
