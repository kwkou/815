

1. 打开项目文件 default.js，并在 **app.OnActivated** 方法重载中，将对 **refreshTodoItems** 方法的最后一个调用替换为以下代码： 
	
        var userId = null;

        // Request authentication from Mobile Services using a Facebook login.
        var login = function () {
            return new WinJS.Promise(function (complete) {
                client.login("facebook").done(function (results) {
                    userId = results.userId;
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

        var authenticate = function () {
            login().then(function () {
                if (userId === null) {

                    // Authentication failed, try again.
                    authenticate();
                }
            });
        }

        authenticate();

    这样可以创建用于存储当前用户的成员变量，以及用于处理身份验证过程的方法。将使用 Facebook 登录对用户进行身份验证。如果使用的标识提供程序不是 Facebook，请将传递给上述 <strong>login</strong> 方法的值更改为下列其中一项：_microsoftaccount_、_twitter_、_google_ 或 _windowsazureactivedirectory_。

    >[AZURE.NOTE]如果你向移动服务注册了 Windows Store 应用软件包信息，则应通过为 <em>useSingleSignOn</em> 参数提供值 <strong>true</strong> 来调用 <a href="http://go.microsoft.com/fwlink/p/?LinkId=322050" target="_blank">login</a> 方法。如果你不执行此操作，则每次调用 login 方法时，系统仍会向你的用户提供登录提示。

2. 按 F5 键运行应用，并使用你选择的标识提供程序登录应用。

   	当你成功登录时，应用应该运行而不出现错误，你应该能够查询移动服务，并对数据进行更新。

<!---HONumber=74-->