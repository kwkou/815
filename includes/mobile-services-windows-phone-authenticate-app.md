1.  打开项目文件 mainpage.xaml.cs 并将以下代码段添加到 MainPage 类：

        private MobileServiceUser user;
        private async System.Threading.Tasks.Task Authenticate()
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

        MessageBox.Show(message);
            }
        }

    这样可以创建用于存储当前用户的成员变量，以及用于处理身份验证过程的方法。将使用 Facebook 登录对用户进行身份验证。

    > [WACOM.NOTE] 如果使用的标识提供者不是 Facebook，请将上述 "MobileServiceAuthenticationProvider" 的值更改为提供者的值。
    >
2.  删除或注释掉现有的 "OnNavigatedTo" 方法覆盖，并将其替换为以下方法，用于处理页的 "Loaded" 事件。

        async void MainPage_Loaded(object sender, RoutedEventArgs e)
        {
        await Authenticate();
        RefreshTodoItems();
        }

    此方法调用新的 "Authenticate" 方法。

3.  将 MainPage 构造函数替换为以下代码：

        // Constructor
        public MainPage()
        {
        InitializeComponent();
        this.Loaded += MainPage_Loaded;
        }

    此构造函数还注册 Loaded 事件的处理程序。

4.  按 F5 键运行应用程序，并使用你选择的标识提供者登录应用程序。

    当你成功登录时，应用程序应该运行而不出现错误，你应该能够查询移动服务，并对数据进行更新。


