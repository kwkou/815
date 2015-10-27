<properties 
	pageTitle="使用 Live Connect 对 Windows 应用商店应用程序进行身份验证" 
	description="了解如何在 Azure 移动服务中从 Windows 应用商店应用程序使用 Live Connect 单一登录。" 
	services="mobile-services" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="08/08/2015" 
	wacn.date="10/22/2015"/>

# 使用 Microsoft 帐户以客户端托管身份验证方式对 Windows 应用商店应用进行身份验证

[AZURE.INCLUDE [mobile-services-selector-single-signon](../includes/mobile-services-selector-single-signon.md)]

##概述
本主题说明如何使用 Live SDK 从通用 Windows Phone 应用获取 Microsoft 帐户的身份验证令牌。然后，你可以使用此令牌在 Azure 移动服务中对用户进行身份验证。在本教程中，你将使用 Live SDK 向现有项目添加 Microsoft 帐户身份验证。成功通过身份验证后，将会欢迎已登录的用户并显示该用户的名称和 ID 值。

>[AZURE.NOTE]本教程将演示使用客户端定向的身份验证和 Live SDK 的好处。这使你可以更轻松地使用移动服务对已登录的用户进行身份验证。你还可以请求额外的范围，使你的应用程序也能访问 OneDrive 等资源。服务定向的身份验证提供更通用的体验，并支持多种身份验证提供程序。有关服务定向的身份验证的详细信息，请参阅[向应用添加身份验证](/documentation/articles/mobile-services-javascript-backend-windows-universal-dotnet-get-started-users)主题。

本教程需要的内容如下：

+ [Live SDK]
+ Microsoft Visual Studio 2013 Update 3 或更高版本。
+ 此外，还必须先完成[移动服务入门](/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started)或[将移动服务添加到现有应用]教程。

##注册应用程序以使用 Microsoft 帐户进行身份验证

若要对用户进行身份验证，必须在 Microsoft 帐户开发人员中心注册你的应用程序。然后，必须将此注册连接到你的移动服务。请完成以下主题中的步骤，以创建 Microsoft 帐户注册并将注册连接到你的移动服务。

+ [注册应用程序以使用 Microsoft 帐户登录](/documentation/articles/mobile-services-how-to-register-microsoft-authentication)

##<a name="permissions"></a>将权限限制给已经过身份验证的用户

接下来，必须限制对资源的访问，在本例中，请确保 *TodoItems* 表只能由登录用户访问。

[AZURE.INCLUDE [mobile-services-restrict-permissions-windows](../includes/mobile-services-restrict-permissions-windows.md)]

##<a name="add-authentication"></a>向应用程序添加身份验证

最后，请添加 Live SDK 并用它来对应用程序中的用户进行身份验证。

1. 在“解决方案资源管理器”中，右键单击该解决方案，然后选择“管理 NuGet 程序包”。

2. 在左窗格中，选择“联机”类别，搜索“LiveSDK”，单击“Live SDK”包对应的“安装”，然后接受许可协议。

  	这会将 Live SDK 添加到解决方案。

3. 打开共享项目文件 MainPage.xaml.cs 并添加以下 using 语句：

        using Microsoft.Live;        

4. 将以下代码段添加到 **MainPage** 类：
	
        private LiveConnectSession session;
        private async System.Threading.Tasks.Task AuthenticateAsync()
        {

            // Get the URL the mobile service.
            var serviceUrl = App.MobileService.ApplicationUri.AbsoluteUri;

            // Create the authentication client using the mobile service URL.
            LiveAuthClient liveIdClient = new LiveAuthClient(serviceUrl);

            while (session == null)
            {
                // Request the authentication token from the Live authentication service.
				// The wl.basic scope is requested.
                LiveLoginResult result = await liveIdClient.LoginAsync(new string[] { "wl.basic" });
                if (result.Status == LiveConnectSessionStatus.Connected)
                {
                    session = result.Session;

                    // Get information about the logged-in user.
                    LiveConnectClient client = new LiveConnectClient(session);
                    LiveOperationResult meResult = await client.GetAsync("me");

                    // Use the Microsoft account auth token to sign in to Mobile Services.
                    MobileServiceUser loginResult = await App.MobileService
                        .LoginWithMicrosoftAccountAsync(result.Session.AuthenticationToken);
	
                    // Display a personalized sign-in greeting.
                    string title = string.Format("Welcome {0}!", meResult.Result["first_name"]);
                    var message = string.Format("You are now logged in - {0}", loginResult.UserId);
                    var dialog = new MessageDialog(message, title);
                    dialog.Commands.Add(new UICommand("OK"));
                    await dialog.ShowAsync();
                }
                else
                {
                    session = null;
                    var dialog = new MessageDialog("You must log in.", "Login Required");
                    dialog.Commands.Add(new UICommand("OK"));
                    await dialog.ShowAsync();
                }
            }
         }

    这样可以创建用于存储当前 Live Connect 会话的成员变量，以及用于处理身份验证过程的方法。

	>[AZURE.NOTE]在从服务请求新令牌之前，你应尝试使用缓存的实时身份验证令牌或移动服务授权令牌。如果你不这样做，则在有许多客户同时尝试启动你的应用时，你可能会遇到与使用量相关的问题。有关如何缓存此令牌的示例，请参阅[身份验证入门](/documentation/articles/mobile-services-windows-store-dotnet-get-started-users/#tokens)
	
5. 注释掉或删除现有 **OnNavigatedTo** 方法覆盖中对 **RefreshTodoItems** 方法的调用。

	这可以防止在对用户进行身份验证之前加载数据。接下来，将要添加一个按钮用于启动登录过程。

6. 将以下代码段添加到 MainPage 类：

        private async void ButtonLogin_Click(object sender, RoutedEventArgs e)
        {
            // Login the user and then load data from the mobile service.
            await AuthenticateAsync();

            // Hide the login button and load items from the mobile service.
            this.ButtonLogin.Visibility = Windows.UI.Xaml.Visibility.Collapsed;
            await RefreshTodoItems();
        }
		
7. 在 Windows Store 应用项目中，打开 MainPage.xaml 项目文件，然后直接在定义“保存”按钮的元素之前添加以下 **Button** 元素：

		<Button Name="ButtonLogin" Click="ButtonLogin_Click" 
                        Visibility="Visible">Sign in</Button>

8. 对 Windows Phone Store 应用项目重复上一步，但这次在 **TitlePanel** 中的 **TextBlock** 元素之后添加 **Button**。
		
9. 按 F5 键运行应用，并使用 Microsoft 帐户登录 Live Connect。

   当你成功登录时，应用应该运行而不出现错误，你应该能够查询移动服务，并对数据进行更新。

10. 右键单击 Windows Phone Store 应用项目，单击“设置为启动项目”，然后重复上一步来验证 Windows Phone Store 应用是否也正常运行。

现在，已由某个注册的标识提供者进行身份验证的任何用户都可访问 *TodoItem* 表。为了进一步提高用户特定数据的安全性，你还必须实施授权。为此，需要获取给定用户的用户 ID，然后根据该 ID 来确定该用户对给定的资源具有哪种级别的访问权限。

## <a name="next-steps"></a>后续步骤

在下一教程[使用脚本为用户授权]中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。有关如何使用其他标识提供者进行身份验证的信息，请参阅[身份验证入门]。请在[移动服务 .NET 操作方法概念性参考]中了解有关如何将移动服务与 .NET 一起使用的详细信息

<!-- Anchors. -->
[Register your app for authentication and configure Mobile Services]: #register
[Restrict table permissions to authenticated users]: #permissions
[Add authentication to the app]: #add-authentication
[Next Steps]: #next-steps

<!-- Images. -->

<!-- URLs. -->
[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039

[将移动服务添加到现有应用]: /documentation/articles/mobile-services-windows-store-dotnet-get-started-data
[身份验证入门]: /documentation/articles/mobile-services-windows-store-dotnet-get-started-users
[使用脚本为用户授权]: /documentation/articles/mobile-services-javascript-backend-service-side-authorization

[Azure Management Portal]: https://manage.windowsazure.cn/
[移动服务 .NET 操作方法概念性参考]: /documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library
[Live SDK]: http://go.microsoft.com/fwlink/p/?LinkId=262253

<!---HONumber=74-->