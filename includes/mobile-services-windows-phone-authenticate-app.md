1. 打开项目文件 mainpage.xaml.cs 并将以下代码段添加到 MainPage 类：
	
        private MobileServiceUser user;
        private async Task Authenticate()
        {
            while (user == null)
            {
                string message;
                try
                {
                    user = await App.MobileServiceDotNetClient.LoginAsync(MobileServiceAuthenticationProvider.Twitter);
                    message = string.Format("You are now logged in - {0}", user.UserId);
                }
                catch (InvalidOperationException)
                {
                    message = "You must log in. Login Required";
                }

                var dialog = new MessageDialog(message);
                await dialog.ShowAsync();
            }
        }

    这样可以创建用于存储当前用户的成员变量，以及用于处理身份验证过程的方法。将使用 Twitter 登录对用户进行身份验证。

    >[AZURE.NOTE]如果使用的标识提供者不是 Twitter，请将上述 **MobileServiceAuthenticationProvider** 的值更改为你的提供者的值。<strong></strong></p></div>

2. 删除或注释掉现有的 **OnNavigatedTo** 方法覆盖，并将其替换为以下方法，用于处理页的 **Loaded** 事件。 

        async void MainPage_Loaded(object sender, RoutedEventArgs e)
        {
            await Authenticate();
            RefreshTodoItems();
        }

   	此方法调用新的 **Authenticate** 方法。 

3. 将 MainPage 构造函数替换为以下代码：

        // Constructor
        public MainPage()
        {
            InitializeComponent();
            this.Loaded += MainPage_Loaded;
        }

   	此构造函数还注册 Loaded 事件的处理程序。
		
4. 按 F5 键运行应用，并使用您选择的标识提供程序登录应用。 

   	当你成功登录时，应用应该运行而不出现错误，你应该能够查询移动服务，并对数据进行更新。

<!---HONumber=Mooncake_0321_2016-->