<properties linkid="develop-mobile-tutorials-single-sign-on-windows-8-dotnet" urlDisplayName="Authenticate with single sign-on" pageTitle="使用 Live Connect 对 Windows 应用商店应用程序进行身份验证" metaKeywords="Azure Live Connect, Azure SSO, SSO Live Connect, mobile services sso, Windows Store app sso" description="了解如何通过 Windows 应用商店应用程序在 Azure 移动服务中使用 Live Connect 单一登录。" metaCanonical="" services="" documentationCenter="" title="Authenticate your Windows Store app with Live Connect single sign-on" authors="glenga" solutions="" manager="" editor="" />



# 使用 Live Connect 单一登录对 Windows 应用商店应用程序进行身份验证
<div class="dev-center-tutorial-selector sublanding"> 
	<a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-single-sign-on/" title="Windows Store C#" class="current">Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/mobile-services-windows-store-javascript-single-sign-on/" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/documentation/articles/mobile-services-windows-phone-single-sign-on/" title="Windows Phone">Windows Phone</a>
</div>	

本主题说明如何使用 Live Connect 单一登录从 Windows 应用商店 8.1 应用程序对 Azure 移动服务的用户进行身份验证。在本教程中，你将使用 Live Connect 向快速入门项目添加身份验证功能。成功通过 Live Connect 进行身份验证后，将使用名称欢迎已登录的用户并显示用户 ID 值。  

>[WACOM.NOTE]本教程演示通过 Live Connect 为 Windows 应用商店应用程序提供单一登录体验的使用好处。这使你可以更轻松地使用移动服务对已登录的用户进行身份验证。有关支持多个身份验证提供程序的更通用的身份验证体验，请参阅主题 <a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-users/">身份验证入门</a>。 

本教程将指导你完成以下启用 Live Connect 身份验证的基本步骤：

1. [注册应用程序以进行身份验证并配置移动服务]
2. [将表权限限制给已经过身份验证的用户]
3. [向应用程序添加身份验证]

本教程需要的内容如下：

+ [Live SDK for Windows]
+ Microsoft Visual Studio 2012 Express for Windows 8 RC 或更高版本
+ 此外，还必须先完成教程[将移动服务添加到现有应用程序]。

## <a name="register"></a>为 Windows 应用商店注册应用程序

为了能够对用户进行身份验证，你必须将应用程序提交到 Windows 应用商店。然后，你必须注册用于将 Live Connect 与移动服务集成的客户端密钥。

[WACOM.INCLUDE [mobile-services-register-windows-store-app](../includes/mobile-services-register-windows-store-app.md)]

## <a name="permissions"></a>将权限限制给已经过身份验证的用户

[WACOM.INCLUDE [mobile-services-restrict-permissions-javascript-backend](../includes/mobile-services-restrict-permissions-javascript-backend.md)] 

<ol start="3">
<li><p>在 Visual Studio 2012 Express for Windows 8 中，打开在完成教程 <a href="/zh-cn/documentation/articles/mobile-services-windows-store-get-started">移动服务入门</a>。</p></li> 
<li><p>按 F5 键运行这个基于快速入门的应用程序；验证启动该应用程序后，是否会引发状态代码为 401（"未授权"）的未处理异常。</p>
   
   	<p>发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 <em>TodoItem</em> 表现在要求身份验证。</p></li>
</ol>

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

## <a name="add-authentication"></a>向应用程序添加身份验证

1. 下载并安装 [Live SDK for Windows]。

2. 在 Visual Studio 的"项目"菜单上，单击"添加引用"，然后展开"Windows"，单击"扩展"，选中 **Live SDK**，然后单击"确定"。 

  	![][16]

  	这将向项目添加 Live SDK 的引用。

5. 打开项目文件 MainPage.xaml.cs 并添加以下 using 语句：

        using Microsoft.Live;        

6. 将以下代码段添加到 MainPage 类：
	
        private LiveConnectSession session;
        private async System.Threading.Tasks.Task AuthenticateAsync()
        {
            LiveAuthClient liveIdClient = new LiveAuthClient("<< INSERT REDIRECT DOMAIN HERE >>");

            while (session == null)
            {
                // Force a logout to make it easier to test with multiple Microsoft Accounts
                if (liveIdClient.CanLogout)
                    liveIdClient.Logout();
	
                LiveLoginResult result = await liveIdClient.LoginAsync(new[] { "wl.basic" });
                if (result.Status == LiveConnectSessionStatus.Connected)
                {
                    session = result.Session;
                    LiveConnectClient client = new LiveConnectClient(result.Session);
                    LiveOperationResult meResult = await client.GetAsync("me");
                    MobileServiceUser loginResult = await App.MobileService
                        .LoginWithMicrosoftAccountAsync(result.Session.AuthenticationToken);
	
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

    这样可以创建用于存储当前 Live Connect 会话的成员变量，以及用于处理身份验证过程的方法。在可能情况下，此代码强制注销以确保每次应用程序运行时都提示用户提供凭据。这样便于使用不同 Microsoft 帐户测试应用程序以确保身份验证正常执行。此机制仅在已登录用户没有已连接的 Microsoft 帐户时正常工作。 

	>[WACOM.NOTE]每次运行应用程序时，不应请求实时连接身份验证令牌或移动服务授权令牌。不仅是因为这样做效率低下，而且，如果许多客户尝试同时启用应用程序，还可能导致与使用相关的问题。较好的方法是缓存令牌，并先尝试使用缓存的移动服务令牌，然后才调用 **LoginWithMicrosoftAccountAsync**。有关如何缓存此令牌的示例，请参阅[身份验证入门](/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-users/#tokens)
	

7. 使用在 Live Connect 中设置应用程序时指定的重定向域（采用 **https://_service-name_.azure-mobile.cn/** 格式）更新上一步中的 _<< INSERT REDIRECT DOMAIN HERE >>_ 字符串。

    <div class="dev-callout"><b>注意</b>
	<p>在 Windows 应用商店应用程序中，通过将重定向域 URI 值传递给类构建器，创建了 <strong>LiveAuthClient</strong> 类的一个实例。在 <a href="/zh-cn/documentation/articles/mobile-services-windows-phone-single-sign-on/">Windows Phone 8 应用程序</a>中，通过传递客户端 ID 将同一个类实例化。</p>
    </div>

8. 将现有 **OnNavigatedTo** 事件处理程序替换为调用新的 **Authenticate** 方法的处理程序：

        protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            await AuthenticateAsync();
            RefreshTodoItems();
        }
		
9. 按 F5 键运行应用程序，并使用 Microsoft 帐户登录 Live Connect。 

   当您成功登录时，应用应该运行而不出现错误，您应该能够查询移动服务，并对数据进行更新。

## <a name="next-steps"> </a>后续步骤

在下一教程[使用脚本为用户授权]中，将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。有关如何使用其他标识提供者进行身份验证的信息，请参阅[身份验证入门]。请在[移动服务 .NET 操作方法概念性参考]中了解有关如何将移动服务与 .NET 一起使用的详细信息。

<!-- Anchors. -->
[注册应用程序以进行身份验证并配置移动服务]: #register
[将表权限限制给已经过身份验证的用户]: #permissions
[向应用程序添加身份验证]: #add-authentication
[后续步骤]:#next-steps

<!-- Images. -->
[0]: ./media/mobile-services-windows-store-dotnet-single-sign-on/mobile-services-submit-win8-app.png
[1]: ./media/mobile-services-windows-store-dotnet-single-sign-on/mobile-services-win8-app-name.png
[2]: ./media/mobile-services-windows-store-dotnet-single-sign-on/mobile-services-store-association.png
[3]: ./media/mobile-services-windows-store-dotnet-single-sign-on/mobile-services-select-app-name.png
[4]: ./media/mobile-services-windows-store-dotnet-single-sign-on/mobile-services-selection.png
[5]: ./media/mobile-services-windows-store-dotnet-single-sign-on/mobile-service-uri.png
[6]: ./media/mobile-services-windows-store-dotnet-single-sign-on/mobile-live-connect-apps-list.png
[7]: ./media/mobile-services-windows-store-dotnet-single-sign-on/mobile-live-connect-app-api-settings.png





[13]: ./media/mobile-services-windows-store-dotnet-single-sign-on/mobile-identity-tab-ma-only.png
[14]: ./media/mobile-services-windows-store-dotnet-single-sign-on/mobile-portal-data-tables.png
[15]: ./media/mobile-services-windows-store-dotnet-single-sign-on/mobile-portal-change-table-perms.png
[16]: ./media/mobile-services-windows-store-dotnet-single-sign-on/mobile-add-reference-live-dotnet.png

<!-- URLs. -->
[提交应用程序页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[我的应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[将移动服务添加到现有应用程序]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-data/
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-users
[使用脚本为用户授权]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-authorize-users-in-scripts/

[Azure 管理门户]: https://manage.windowsazure.cn/
[移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library
