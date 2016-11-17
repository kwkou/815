<properties
	pageTitle="Azure AD Windows 应用商店入门 | Azure"
	description="如何生成一个与 Azure AD 集成以方便登录，并使用 OAuth 调用 Azure AD 保护 API 的 Windows 应用商店应用程序。"
	services="active-directory"
	documentationCenter="windows"
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="mobile-windows-store"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="09/16/2016"
	wacn.date="10/17/2016"
	ms.author="dastrock"/>  



# 将 Azure AD 与 Windows 应用商店应用集成

[AZURE.INCLUDE [active-directory-devquickstarts-switcher](../../includes/active-directory-devquickstarts-switcher.md)]

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

如果你要开发 Windows 应用商店应用程序，Azure AD 可让你简单直接地使用用户的 Active Directory 帐户对其进行身份验证。它还可以让应用程序安全地使用 Azure AD 保护的任何 Web API，例如 Office 365 API 或 Azure API。

对于需要访问受保护资源的 Windows 应用商店桌面应用程序，Azure AD 提供 Active Directory 身份验证库 (ADAL)。在本质上，ADAL 的唯一用途就是方便应用程序获取访问令牌。为了演示操作的简单性，下面我们要生成一个“目录搜索器”Windows 应用商店应用程序，该应用程序可以：

-	使用 [OAuth 2.0 身份验证协议](/documentation/articles/active-directory-protocols-oauth-code/)获取调用 Azure AD Graph API 的访问令牌。
-	在目录中搜索具有给定 UPN 的用户。
-	将用户注销。

若要生成完整的工作应用程序，你需要：

2. 将应用程序注册到 Azure AD。
3. 安装并配置 ADAL。
5. 使用 ADAL 从 Azure AD 获取令牌。

若要开始，请[下载框架项目](https://github.com/AzureADQuickStarts/NativeClient-WindowsStore/archive/skeleton.zip)或[下载已完成的示例](https://github.com/AzureADQuickStarts/NativeClient-WindowsStore/archive/complete.zip)。每个下载项目都是 Visual Studio 2015 解决方案。你还需要一个可在其中创建用户和注册应用程序的 Azure AD 租户。如果你还没有租户，请[了解如何获取租户](/documentation/articles/active-directory-howto-tenant/)。

## *1.注册目录搜索器应用程序*
若要让应用程序获取令牌，首先需要在 Azure AD 租户中注册该应用程序，并授予它访问 Azure AD Graph API 的权限：

-	登录到 [Azure 管理门户](https://manage.windowsazure.cn)
-	在左侧的导航栏中单击“Active Directory”
-	选择要在其中注册应用程序的租户。
-	单击“应用程序”选项卡，然后在底部抽屉中单击“添加”。
-	根据提示创建一个新的“本机客户端应用程序”。
    -	应用程序的“名称”向最终用户描述你的应用程序
    -	“重定向 URI”是 Azure AD 要用来返回令牌响应的方案与字符串组合。暂时输入一个占位符值，例如 `http://DirectorySearcher`。稍后我们将会替换此值。
-	完成注册后，AAD 将为应用程序分配唯一的客户端标识符。在后面的部分中将会用到此值，因此，请从“配置”选项卡复制此值。
- 另外，请在“配置”选项卡中，找到“针对其他应用程序的权限”部分。对于“Azure Active Directory”应用程序，在“委托的权限”下添加“以已登录用户的身份访问目录”权限。这样，你的应用程序便可以在 Graph API 中查询用户。

## *2.安装并配置 ADAL*
将应用程序注册到 Azure AD 后，可以安装 ADAL 并编写标识相关的代码。为了使 ADAL 能够与 Azure AD 通信，需要为 ADAL 提供一些有关应用程序的注册信息。
-	首先，使用 Package Manager Console 将 ADAL 添加到 DirectorySearcher 项目。


	PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory


-	在 DirectorySearcher 项目中，打开 `MainPage.xaml.cs`。替换 `Config Values` 区域中的值，以反映你在 Azure 门户中输入的值。只要使用 ADAL，你的代码就会引用这些值。
    -	`tenant` 是 Azure AD 租户的域，例如 contoso.partner.onmschina.cn
    -	`clientId` 是从门户复制的应用程序 clientId。
-	你现在需要发现 Windows 应用商店应用的回叫 URI。在 `MainPage` 方法中的此行上设置一个断点：


	redirectURI = Windows.Security.Authentication.Web.WebAuthenticationBroker.GetCurrentApplicationCallbackUri();

- 生成解决方案，并确保还原所有程序包引用。如果缺少程序包，请打开 Nuget 程序包管理器并还原程序包。
- 运行应用程序，并在到达断点时，将 `redirectUri` 的值复制到单独的位置。该值应类似于


	ms-app://s-1-15-2-1352796503-54529114-405753024-3540103335-3203256200-511895534-1429095407/


- 返回到 Azure 管理门户中应用程序的“配置”选项卡，并将 **RedirectUri** 的值替换为此值。

## *3.使用 ADAL 从 Azure AD 获取令牌*
ADAL 遵守的基本原理是，每当应用程序需要访问令牌时，它只需调用 `authContext.AcquireToken(…)`，然后 ADAL 就会负责其余的工作。

-	第一步是初始化应用程序的 `AuthenticationContext`（ADAL 的主类）。你将在此处传递 ADAL 与 Azure AD 通信时所需的坐标，并告诉 ADAL 如何缓存令牌。

C#
		
	public MainPage()
	{
	    ...

	    authContext = new AuthenticationContext(authority);
	}
		

- 现在查找 `Search(...)` 方法，当用户在应用的 UI 中单击“搜索”按钮时，将调用该方法。此方法将向 Azure AD Graph API 发出 GET 请求，以查询其 UPN 以给定搜索词开头的用户。但是，若要查询 Graph API，你需要在请求的 `Authorization` 标头中包含 access\_token - 这是 ADAL 传入的位置。

C#

	private async void Search(object sender, RoutedEventArgs e)
	{
	    ...
	    AuthenticationResult result = null;
	    try
	    {
	        result = await authContext.AcquireTokenAsync(graphResourceId, clientId, redirectURI, new PlatformParameters(PromptBehavior.Auto, false));
	    }
	    catch (AdalException ex)
	    {
	        if (ex.ErrorCode != "authentication_canceled")
	        {
	            ShowAuthError(string.Format("If the error continues, please contact your administrator.\n\nError: {0}\n\nError Description:\n\n{1}", ex.ErrorCode, ex.Message));
	        }
	        return;
	    }
	    ...
	}

- 当应用程序通过调用 `AcquireTokenAsync(...)` 请求令牌时，ADAL 将尝试返回一个令牌，而不要求用户输入凭据。如果 ADAL 确定用户需要登录以获取令牌，将显示登录对话框，收集用户的凭据，并在身份验证成功后返回令牌。如果 ADAL 出于任何原因无法返回令牌，`AuthenticationResult` 将处于错误状态。

- 现在，你可以使用刚刚获得的 access\_token。同时，在 `Search(...)` 方法中，将令牌附加到 Authorization 标头的 Graph API GET 请求中：

C#
		
	// Add the access token to the Authorization Header of the call to the Graph API, and call the Graph API.
	httpClient.DefaultRequestHeaders.Authorization = new HttpCredentialsHeaderValue("Bearer", result.AccessToken);


- 你还可以使用 `AuthenticationResult` 对象在应用程序中显示有关用户的信息，例如，用户的 ID：

C#

	// Update the Page UI to represent the signed in user
	ActiveUser.Text = result.UserInfo.DisplayableId;

- 最后，还可以使用 ADAL 将用户从应用程序中注销。当用户单击“注销”按钮时，我们希望确保 `AcquireTokenAsync(...)` 的后续调用会显示登录视图。使用 ADAL 时，只需清除令牌缓存即可：

C#

	private void SignOut()
	{
	    // Clear session state from the token cache.
	    authContext.TokenCache.Clear();

	    ...
	}


祝贺你！ 现在，你已创建一个有效的 Windows 应用商店应用程序，它可以对用户进行身份验证，使用 OAuth 2.0 安全调用 Web API，并获取有关用户的基本信息。如果你尚未这样做，可以在租户中填充一些用户。运行你的 DirectorySearcher 应用程序，并使用这些用户之一进行登录。根据用户的 UPN 搜索其他用户。关闭应用程序，然后重新运行它。请注意，用户的会话将保持不变。注销（右键单击以显示底部栏），然后以另一用户的身份重新登录。

使用 ADAL 可以方便地将所有这些常见标识功能合并到应用程序中。它会负责所有的繁琐工作 - 缓存管理、OAuth 协议支持、向用户显示登录名 UI、刷新已过期的令牌，等等。你只需要真正了解一个 API 调用，即 `authContext.AcquireToken*(…)`。

[此处](https://github.com/AzureADQuickStarts/NativeClient-WindowsStore/archive/complete.zip)提供了已完成示例（无需配置值）供你参考。现在，你可以转到其他标识方案。你可能想要尝试：

[使用 Azure AD 保护 .NET Web API >>](/documentation/articles/active-directory-devquickstarts-webapi-dotnet/)

[AZURE.INCLUDE [active-directory-devquickstarts-additional-resources](../../includes/active-directory-devquickstarts-additional-resources.md)]

<!---HONumber=Mooncake_1010_2016-->
