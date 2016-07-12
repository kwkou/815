<properties
	pageTitle="使用 Active Directory 身份验证库单一登录对应用进行身份验证（Windows 应用商店）| Azure"
	description="了解如何在 Windows 应用商店应用程序中使用 ADAL 对用户进行单一登录身份验证。"
	documentationCenter="windows"
	authors="wesmc7777"
	manager="dwrede"
	editor=""
	services="mobile-services"/>

<tags 
	ms.service="mobile-services" 
	ms.date="01/14/2016"
	wacn.date="05/23/2016"/>

# 使用 Active Directory 身份验证库单一登录对应用程序进行身份验证


[AZURE.INCLUDE [mobile-services-selector-adal-sso](../includes/mobile-services-selector-adal-sso.md)]

##概述

在本教程中，你将使用 Active Directory 身份验证库将身份验证添加到快速入门项目，以支持使用 Azure Active Directory 进行[客户端定向的登录操作](http://msdn.microsoft.com/zh-cn/library/azure/jj710106.aspx)。若要支持使用 Azure Active Directory 进行[服务定向的登录操作](http://msdn.microsoft.com/zh-cn/library/azure/dn283952.aspx)，请从[向移动服务应用添加身份验证](/documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-users/)教程入手。

若要能够对用户进行身份验证，必须向 Azure Active Directory (AAD) 注册你的应用程序。此过程分为两个步骤。首先，你必须注册你的移动服务，并公开其上的权限。其次，你必须注册你的 Windows 应用商店应用程序，并授予它对这些权限的访问权限


>[AZURE.NOTE]本教程旨在帮助你更好地了解如何通过移动服务，使用[客户端定向的登录操作](http://msdn.microsoft.com/zh-cn/library/azure/jj710106.aspx)对 Windows 应用商店应用进行单一登录 Azure Active Directory 身份验证。如果这是你第一次体验移动服务，请先完成[移动服务入门]教程。


##先决条件

本教程需要的内容如下：

* 在 Windows 8.1 上运行的 Visual Studio 2013。
* 完成[移动服务入门]教程。
* Microsoft Azure 移动服务 SDK NuGet 包
* Active Directory 身份验证库 NuGet 包 

[AZURE.INCLUDE [mobile-services-dotnet-adal-register-service](../includes/mobile-services-dotnet-adal-register-service.md)]

##向 Azure Active Directory 注册你的应用程序

若要向 Azure Active Directory 注册应用程序，必须将该应用程序关联到 Windows 应用商店，并为该应用程序提供程序包安全标识符 (SID)。在 Azure Active Directory 中使用本机应用程序设置注册该程序包 SID。


### 将该应用程序与新的应用商店应用程序名称相关联

1. 在 Visual Studio 中，右键单击客户端应用项目，单击“应用商店”，然后单击“将应用与应用商店关联”

    ![][1]

2. 登录到你的开发人员中心帐户。

3. 输入要为该应用保留的应用名称，然后单击“保留”。

    ![][2]

4. 选择新的应用名称，并单击“下一步”。

5. 单击“关联”将该应用与应用商店名称相关联。


### 检索应用程序的程序包 SID

现在，你需要检索使用本机应用程序设置配置的程序包 SID。

1. 登录到 [Windows 开发人员中心仪表板]，并在该应用上单击“编辑”。

    ![][3]

2. 然后单击应用管理“>应用标识”，然后复制页面中的包 SID。

    ![][4]


###创建本机应用程序注册

1. 在[Azure 管理门户]中浏览到“Active Directory”，然后单击你的目录。

    ![][7]

2. 单击顶部的“应用程序”选项卡，然后单击“添加”以添加应用程序。

    ![][8]

3. 单击“添加我的组织正在开发的应用程序”。

4. 在“添加应用程序”向导中，为应用程序输入**名称**，并单击“本机客户端应用程序”类型。然后单击以继续。

    ![][9]

5. 在“重定向 URI”框中，粘贴先前复制的应用程序包 SID，然后单击以完成本机应用注册。

    ![][10]

6. 单击本机应用程序的“配置”选项卡，并复制“客户端 ID”。稍后你将需要此项。

    ![][11]

7. 将页面向下滚动到“其他应用程序的权限”部分，并为先前注册的移动服务应用程序授予完全访问权限。然后单击“保存”

    ![][12]

现在，将在 AAD 中配置你的移动服务，以接收你的应用程序发出的单一登录请求。



##将移动服务配置为要求身份验证

[AZURE.INCLUDE [mobile-services-restrict-permissions-dotnet-backend](../includes/mobile-services-restrict-permissions-dotnet-backend.md)]

##向客户端应用程序添加身份验证代码

1. 在 Visual Studio 中打开 Windows 应用商店客户端应用程序项目。

[AZURE.INCLUDE [mobile-services-dotnet-adal-install-nuget](../includes/mobile-services-dotnet-adal-install-nuget.md)]

4. 在 Visual Studio 的“解决方案资源管理器”窗口中，打开 MainPage.cs 文件，并添加以下 using 语句。

        using Windows.UI.Popups;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;
        using Newtonsoft.Json.Linq;


5. 将以下代码添加到声明了 `AuthenticateAsync` 方法的 MainPage 类。

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
                  AuthenticationContext ac = new AuthenticationContext(authority);
                  AuthenticationResult ar = await ac.AcquireTokenAsync(resourceURI, clientID, (Uri) null);
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

6. 在上面的 `AuthenticateAsync` 方法的代码中，将 **INSERT-AUTHORITY-HERE** 替换为在其中进行应用程序设置的租户的名称，格式应为 https://login.chinacloudapi.cn/tenant-name.onmicrosoft.com。 可以在 [Azure 管理门户]中从 Azure Active Directory 的“域”选项卡复制此值。

7. 在上面的 `AuthenticateAsync` 方法的代码中，将 **INSERT-RESOURCE-URI-HERE** 替换为你的移动服务的“应用程序 ID URI”。如果你按照[如何向 Azure Active Directory 注册]主题进行操作，你的应用程序 ID URI 应该类似于 https://todolist.azure-mobile.net/login/aad 。

8. 在上面的 `AuthenticateAsync` 方法的代码中，将 **INSERT-CLIENT-ID-HERE** 替换为你从本机客户端应用程序复制的客户端 ID。

9. 在 Visual Studio 的“解决方案资源管理器”窗口中，打开客户端项目中的 Package.appxmanifest 文件。单击“功能”选项卡，并启用“企业应用程序”和“私有网络(客户端和服务器)”。保存文件。

    ![][14]

10. 在 MainPage.cs 文件中，更新 `OnNavigatedTo` 事件处理程序以便按如下所示调用 `AuthenticateAsync` 方法。

        protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            await AuthenticateAsync();
            await RefreshTodoItems();
        }


##测试使用身份验证的客户端

1. 在 Visual Studio 中，运行客户端应用程序。
2. 您将收到登录 Azure Active Directory 的提示。  
3. 该应用程序将进行身份验证并返回 Todo 项目。

    ![][15]




<!-- Images -->
[0]: ./media/mobile-services-windows-store-dotnet-adal-sso-authenticate/mobile-services-aad-app-manage-manifest.png
[1]: ./media/mobile-services-windows-store-dotnet-adal-sso-authentication/mobile-services-vs-associate-app.png
[2]: ./media/mobile-services-windows-store-dotnet-adal-sso-authentication/mobile-services-vs-reserve-store-appname.png
[3]: ./media/mobile-services-windows-store-dotnet-adal-sso-authentication/mobile-services-store-app-edit.png
[4]: ./media/mobile-services-windows-store-dotnet-adal-sso-authentication/mobile-services-store-app-services.png
[5]: ./media/mobile-services-windows-store-dotnet-adal-sso-authentication/mobile-services-live-services-site.png
[6]: ./media/mobile-services-windows-store-dotnet-adal-sso-authentication/mobile-services-store-app-package-sid.png
[7]: ./media/mobile-services-windows-store-dotnet-adal-sso-authentication/mobile-services-select-aad.png
[8]: ./media/mobile-services-windows-store-dotnet-adal-sso-authentication/mobile-services-aad-applications-tab.png
[9]: ./media/mobile-services-windows-store-dotnet-adal-sso-authentication/mobile-services-native-selection.png
[10]: ./media/mobile-services-windows-store-dotnet-adal-sso-authentication/mobile-services-native-sid-redirect-uri.png
[11]: ./media/mobile-services-windows-store-dotnet-adal-sso-authentication/mobile-services-native-client-id.png
[12]: ./media/mobile-services-windows-store-dotnet-adal-sso-authentication/mobile-services-native-add-permissions.png
[14]: ./media/mobile-services-windows-store-dotnet-adal-sso-authentication/mobile-services-package-appxmanifest.png
[15]: ./media/mobile-services-windows-store-dotnet-adal-sso-authentication/mobile-services-app-run.png

<!-- URLs. -->
[如何向 Azure Active Directory 注册]: /documentation/articles/mobile-services-how-to-register-active-directory-authentication/
[Azure 管理门户]: https://manage.windowsazure.cn/

[移动服务入门]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started/
[Windows 开发人员中心仪表板]: http://go.microsoft.com/fwlink/p/?LinkID=266734

<!---HONumber=Mooncake_0118_2016-->