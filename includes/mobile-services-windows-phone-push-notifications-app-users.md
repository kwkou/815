
接下来，必须更改推送通知的注册方式，以确保在尝试注册之前对用户进行身份验证。 

1. 在 Visual Studio 的 Solution Explorer 中，打开 app.xaml.cs 项目文件，然后在 **Application_Launching** 事件处理程序中，注释掉或删除对 **AcquirePushChannel** 方法的调用。 
 
2. 将 **AcquirePushChannel** 方法的可访问性从 `私有`更改为 `公共`然后添加 `静态`修饰符。 

3. 打开 MainPage.xaml.cs 项目文件，并将 **OnNavigatedTo** 方法覆盖替换为以下项：

	    protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            await AuthenticateAsync();            
            App.AcquirePushChannel();
            RefreshTodoItems();
        }
