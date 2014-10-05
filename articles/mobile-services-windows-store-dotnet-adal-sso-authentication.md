<properties linkid="develop-mobile-tutorials-sso-with-adal" urlDisplayName="Active Directory SSO Authentication with ADAL" pageTitle="Authenticate your app with Active Directory Authentication Library Single Sign-On (Windows Store) | Mobile Dev Center" metaKeywords="" description="Learn how to authentication users for single sign-on with ADAL in your Windows Store application." metaCanonical="" disqusComments="1" umbracoNaviHide="1" documentationCenter="Mobile" title="Authenticate your app with Active Directory Authentication Library Single Sign-On" authors="wesmc" />

# 使用 Active Directory 身份验证库单一登录对应用程序进行身份验证

<div class="dev-center-tutorial-selector sublanding">
<a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-adal-sso-authentication" title="Windows Store C#" class="current">Windows 应用商店 C\#</a>
</div>

在本教程中，你将使用 Active Directory 身份验证库向快速入门项目添加身份验证功能。

若要能够对用户进行身份验证，必须向 Azure Active Directory (AAD) 注册你的应用程序。此过程分为两个步骤。首先，你必须注册你的移动服务，并公开其上的权限。其次，你必须注册你的 Windows 应用商店应用程序，并授予它对这些权限的访问权限

> [WACOM.NOTE] 本教程旨在帮助你更好地了解如何使用移动服务对 Windows 应用商店应用程序进行单一登录 Azure Active Directory 身份验证。如果这是你第一次体验移动服务，请先完成[移动服务入门][]教程。

本教程将指导你完成以下基本步骤：

1.  [向 Azure Active Directory 注册你的移动服务][]
2.  [向 Azure Active Directory 注册你的应用程序][]
3.  [将移动服务配置为要求身份验证][]
4.  [向客户端应用程序添加身份验证代码][]
5.  [测试使用身份验证的客户端][]

本教程需要的内容如下：

-   运行在 Windows 8.1 上的 Visual Studio 2013。
-   完成[移动服务入门][]或[数据入门][]教程。
-   Windows Azure 移动服务 SDK NuGet 包
-   Active Directory 身份验证库 NuGet 包

<a name="register-mobile-service-aad"></a>
## 向 Azure Active Directory 注册你的移动服务

在本节中，你将向 Azure Active Directory 注册你的移动服务，并配置权限以允许单一登录模拟。

1.  按照[如何向 Azure Active Directory 注册][]主题所述，向 Azure Active Directory 注册你的应用程序。

2.  在 [Azure 管理门户][]中，返回到 Azure Active Directory 扩展，并单击你的活动目录

3.  单击“应用程序” 选项卡，然后单击你的应用程序。

4.  单击“管理清单” 。然后单击“下载清单” ，并将应用程序清单保存到本地目录中。

    ![][]

5.  使用 Visual Studio 打开应用程序清单文件。在文件顶部找到应用程序权限行，该行如下所示：

        "appPermissions": [],

    将该行替换为以下应用程序权限，并保存该文件。

        "appPermissions": [
            {
        "claimValue":"user_impersonation",
        "description":"Allow the application access to the mobile service",
        "directAccessGrantTypes": [],
        "displayName":"Have full access to the mobile service",
        "impersonationAccessGrantTypes": [
                    {
        "impersonated":"User",
        "impersonator":"Application"
                    }
                ],
        "isDisabled":false,
        "origin":"Application",
        "permissionId":"b69ee3c9-c40d-4f2a-ac80-961cd1534e40",
        "resourceScopeType":"Personal",
        "userConsentDescription":"Allow the application full access to the mobile service on your behalf",
        "userConsentDisplayName":"Have full access to the mobile service"
            }
        ],

6.  在 Azure 管理门户中，再次针对应用程序单击“管理清单” ，然后单击“上载清单” 。浏览到你刚更新的应用程序清单的位置，然后上载该清单。

<a name="register-app-aad"></a>
## 向 Azure Active Directory 注册你的应用程序

若要向 Azure Active Directory 注册应用程序，必须将该应用程序关联到 Windows 应用商店，并为该应用程序提供程序包安全标识符 (SID)。在 Azure Active Directory 中使用本机应用程序设置注册该程序包 SID。

### 将该应用程序与新的应用商店应用程序名称相关联

1.  在 Visual Studio 中，右键单击客户端应用程序项目，单击“应用商店” ，然后单击“将应用程序与应用商店关联” 

    ![][1]

2.  登录到你的开发人员中心帐户。

3.  输入要为该应用程序保留的应用程序名称，然后单击“保留” 。

    ![][2]

4.  选择新的应用程序名称，并单击“下一步” 。

5.  单击“关联” 将该应用程序与应用商店名称相关联。

### 检索应用程序的程序包 SID。

现在，你需要检索使用本机应用程序设置配置的程序包 SID。

1.  登录到 [Windows 开发人员中心仪表板][]，并在该应用程序上单击“编辑” 。

    ![][3]

2.  然后单击“服务” 

    ![][4]

3.  然后单击“Live 服务站点” 。

    ![][5]

4.  从页面顶部复制程序包 SID。

    ![][6]

### 创建本机应用程序注册

1.  导航到 [Azure 管理门户][]中的“Active Directory” ，然后单击你的目录。

    ![][7]

2.  单击顶部的“应用程序” 选项卡，然后单击“添加” 以添加应用程序。

    ![][8]

3.  单击“添加我的组织正在开发的应用程序” 。

4.  在“添加应用程序”向导中，为应用程序输入“名称” ，并单击“本机客户端应用程序” 类型。然后单击以继续。

    ![][9]

5.  在“重定向 URI” 框中，粘贴先前复制的应用程序包 SID，然后单击以完成本机应用程序注册。

    ![][10]

6.  单击本机应用程序所对应的“配置” 选项卡，并复制“客户端 ID” 。稍后你将需要此项。

    ![][11]

7.  将页面向下滚动到“其他应用程序的权限” 部分，并为先前注册的移动服务应用程序授予完全访问权限。然后，单击“保存” 

    ![][12]

现在，将在 AAD 中配置你的移动服务，以接收你的应用程序发出的单一登录请求。

<a name="require-authentication"></a>
## 将移动服务配置为要求身份验证

将 .NET 后端移动服务配置为要求身份验证。

1.  在 Visual Studio 中打开 .NET 后端移动服务项目。

2.  在 Visual Studio 的“解决方案资源管理器”窗口中，展开 .NET 后端移动服务项目下的“Controllers” 文件夹。打开 TodoItemController.cs 文件并添加以下 using 语句

        using Microsoft.WindowsAzure.Mobile.Service.Security;

3.  在 TodoItemController.cs 文件中，将 `[AuthorizeLevel(AuthorizationLevel.User)]` 特性添加到以下每个方法，并保存 TodoItemController.cs 文件。

        [AuthorizeLevel(AuthorizationLevel.User)]
        public IQueryable<TodoItem> GetAllTodoItems()
        ...
        [AuthorizeLevel(AuthorizationLevel.User)]
        public SingleResult<TodoItem> GetTodoItem(string id)
        ...
        [AuthorizeLevel(AuthorizationLevel.User)]
        public Task<TodoItem> PatchTodoItem(string id, Delta<TodoItem> patch)
        ...
        [AuthorizeLevel(AuthorizationLevel.User)]
        public async Task<IHttpActionResult> PostTodoItem(TodoItem item)
        ...
        [AuthorizeLevel(AuthorizationLevel.User)]
        public Task DeleteTodoItem(string id)
        ...

4.  右键单击 .NET 后端移动服务项目，然后单击“重新生成” 。
5.  右键单击 .NET 后端移动服务项目，然后单击“发布” ，将你的更改发布到 Azure 帐户。

<a name="add-authentication-code"></a>
## 向客户端应用程序添加身份验证代码

1.  在 Visual Studio 中打开 Windows 应用商店客户端应用程序项目。

2.  在 Visual Studio 的“解决方案资源管理器”窗口中，右键单击客户端应用程序项目，然后单击“管理 NuGet 包” 。

3.  在 NuGet 包管理器中，单击“联机” 和“包括预发行版” 。输入“Microsoft.IdentityModel.Clients” 作为搜索词。然后，单击“安装” 以安装 Active Directory 身份验证库 Nuget 包。

    ![][13]

4.  在 Visual Studio 的“解决方案资源管理器”窗口中，打开 MainPage.xaml.cs 文件，并添加以下 using 语句。

        using Windows.UI.Popups;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;
        using Newtonsoft.Json.Linq;

5.  将以下代码添加到声明了 `AuthenticateAsync` 方法的 MainPage 类。

        private MobileServiceUser user; 
        private async Task AuthenticateAsync()
        {
        string authority = "<INSERT-AUTHORITY-HERE>";
        string resourceURI = "<INSERT-RESOURCE-URI-HERE>";
        string clientID = "<INSERT-CLIENT-ID-HERE>"; 
        while (user == null)
            {
        string message;
        try
                {
        AuthenticationContext ac = new AuthenticationContext(authority, false);
        AuthenticationResult ar = await ac.AcquireTokenAsync(resourceURI, clientID);
        JObject payload = new JObject();
        payload["access_token"] = ar.AccessToken;
        user = await App.MobileService.LoginAsync(MobileServiceAuthenticationProvider.WindowsAzureActiveDirectory, payload);
        message = string.Format("You are now logged in - {0}", user.UserId);
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

6.  在上面的 `AuthenticateAsync` 方法的代码中，将 "INSERT-AUTHORITY-HERE" 替换为在其中进行应用程序设置的租户的名称，格式应为 <https://login.windows.net/tenant-name.onmicrosoft.com>。可以在 [Azure 管理门户][]中从 Azure Active Directory 的“域”选项卡复制出此值。

7.  在上面的 `AuthenticateAsync` 方法的代码中，将 "INSERT-RESOURCE-URI-HERE" 替换为你的移动服务的“应用程序 ID URI” 。如果你按照[如何向 Azure Active Directory 注册][]主题进行操作，你的应用程序 ID URI 应该类似于 <https://todolist.azure-mobile.net/login/aad>。

8.  在上面的 `AuthenticateAsync` 方法的代码中，将 "INSERT-CLIENT-ID-HERE" 替换为你从本机客户端应用程序复制的客户端 ID。

9.  在 Visual Studio 的“解决方案资源管理器”窗口中，打开客户端项目中的 Package.appxmanifest 文件。单击“功能” 选项卡，并启用“企业应用程序” 和“私有网络(客户端和服务器)” 。保存文件。

    ![][14]

10. 在 MainPage.cs 文件中，更新 `OnNavigatedTo` 事件处理程序以调用 `AuthenticateAsync` 方法，如下所示。

        protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
        if (user == null)
        await AuthenticateAsync();

        RefreshTodoItems();
        }

<a name="test-client"></a>
## 测试使用身份验证的客户端

1.  在 Visual Studio 中，运行客户端应用程序。
2.  你将收到要你登录 Azure Active Directory 的提示。
3.  该应用程序将进行身份验证并返回 Todo 项目。

    ![][15]

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-adal-sso-authentication "Windows 应用商店 C#"
  [移动服务入门]: /zh-cn/documentation/articles/mobile-services-windows-store-get-started/
  [向 Azure Active Directory 注册你的移动服务]: #register-mobile-service-aad
  [向 Azure Active Directory 注册你的应用程序]: #register-app-aad
  [将移动服务配置为要求身份验证]: #require-authentication
  [向客户端应用程序添加身份验证代码]: #add-authentication-code
  [测试使用身份验证的客户端]: #test-client
  [数据入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/
  [如何向 Azure Active Directory 注册]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/
  [Azure 管理门户]: https://manage.windowsazure.cn/
  []: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-aad-app-manage-manifest.png
  [1]: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-vs-associate-app.png
  [2]: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-vs-reserve-store-appname.png
  [Windows 开发人员中心仪表板]: http://go.microsoft.com/fwlink/p/?LinkID=266734
  [3]: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-store-app-edit.png
  [4]: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-store-app-services.png
  [5]: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-live-services-site.png
  [6]: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-store-app-package-sid.png
  [7]: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-select-aad.png
  [8]: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-aad-applications-tab.png
  [9]: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-native-selection.png
  [10]: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-native-sid-redirect-uri.png
  [11]: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-native-client-id.png
  [12]: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-native-add-permissions.png
  [13]: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-adal-nuget-package.png
  [14]: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-package-appxmanifest.png
  [15]: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-app-run.png
