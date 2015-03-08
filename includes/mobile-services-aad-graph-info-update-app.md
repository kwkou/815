1. 在 Visual Studio 中，打开 MainPage.xaml.cs 并将以下  `using` 语句添加到文件顶部。
 
        using System.Net.Http;

2. 在 MainPage.xaml.cs 中，将以下类定义添加到名称空间以帮助序列化用户信息。

        public class UserInfo
        {
            public String displayName { get; set; }
            public String streetAddress { get; set; }
            public String city { get; set; }
            public String state { get; set; }
            public String postalCode { get; set; }
        }


3. 在 MainPage.xaml.cs 中，更新  `AuthenticateAsync` 方法来调用自定义 API，从 AAD 返回有关用户的其他信息。 

        private async System.Threading.Tasks.Task AuthenticateAsync()
        {
            while (user == null)
            {
                string message;
                try
                {
                    user = await App.MobileService
                        .LoginAsync(MobileServiceAuthenticationProvider.WindowsAzureActiveDirectory);
                    UserInfo userInfo = await App.MobileService.InvokeApiAsync<UserInfo>("getuserinfo", 
                        HttpMethod.Get, null);
                    message = string.Format("{0}, you are now logged in.\n\nYour address is...\n\n{1}\n{2}, {3} {4}", 
                        userInfo.displayName, userInfo.streetAddress, userInfo.city, userInfo.state, 
                        userInfo.postalCode);
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


4. 保存您的更改并构建该服务以验证是否没有语法错误。  
