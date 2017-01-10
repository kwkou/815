<properties
	pageTitle="Azure AD Xamarin 入门 | Azure"
	description="如何生成一个与 Azure AD 集成以方便登录，并使用 OAuth 调用 Azure AD 保护 API 的 Xamarin 应用程序。"
	services="active-directory"
	documentationCenter="xamarin"
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="mobile-xamarin"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="dastrock"
   	wacn.date="01/09/2017"/>  



# 将 Azure AD 集成到 Xamarin 应用程序中

[AZURE.INCLUDE [active-directory-devquickstarts-switcher](../../includes/active-directory-devquickstarts-switcher.md)]

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

Xamarin 允许你使用 C# 编写可在 iOS、Android 和 Windows（移动设备和电脑）上运行的移动应用。如果你要使用 Xamarin 生成应用程序，Azure AD 可让你简单直接地使用用户的 Active Directory 帐户对其进行身份验证。它还可以让应用程序安全地使用 Azure AD 保护的任何 Web API，例如 Office 365 API 或 Azure API。

对于需要访问受保护资源的 Xamarin 应用程序，Azure AD 提供 Active Directory 身份验证库 (ADAL)。在本质上，ADAL 的唯一用途就是方便应用程序获取访问令牌。为了演示操作的简单性，下面我们要生成一个“目录搜索器”应用程序，该应用程序可以：

-	在 iOS、Android、Windows 桌面、Windows Phone 和 Windows 应用商店上运行。
- 使用单个可移植类库 (PCL) 对用户进行身份验证，并获取 Azure AD Graph API 的令牌
-	在目录中搜索具有给定 UPN 的用户。

若要生成完整的工作应用程序，你需要：

1. 设置 Xamarin 开发环境
2. 将应用程序注册到 Azure AD。
3. 安装并配置 ADAL。
4. 使用 ADAL 从 Azure AD 获取令牌。

若要开始，请[下载框架项目](https://github.com/AzureADQuickStarts/NativeClient-MultiTarget-DotNet/archive/skeleton.zip)或[下载已完成的示例](https://github.com/AzureADQuickStarts/NativeClient-MultiTarget-DotNet/archive/complete.zip)。每个下载项目都是 Visual Studio 2013 解决方案。你还需要一个可在其中创建用户和注册应用程序的 Azure AD 租户。如果你还没有租户，请[了解如何获取租户](/documentation/articles/active-directory-howto-tenant/)。

## *0.设置 Xamarin 开发环境*
因为本教程包含 iOS、Android 和 Windows 项目，所以你需要 Visual Studio 和 Xamarin。若要创建必要的环境，请遵循 MSDN 上 [Visual Studio 和 Xamarin 的设置和安装](https://msdn.microsoft.com/zh-cn/library/mt613162.aspx)中的完整说明。这些说明包含的材料可供你在等待安装程序完成时查看，以深入了解 Xamarin。

完成必要的设置后，在 Visual Studio 中打开解决方案以开始操作。你会看到六个项目：五个特定于平台的项目，一个要在所有平台之间共享的可移植类库，即 `DirectorySearcher.cs`

## *1.注册目录搜索器应用程序*
若要让应用程序获取令牌，首先需要在 Azure AD 租户中注册该应用程序，并授予它访问 Azure AD Graph API 的权限：

-	登录到 [Azure 管理门户](https://manage.windowsazure.cn)
-	在左侧的导航栏中单击“Active Directory”
-	选择要在其中注册应用程序的租户。
-	单击“应用程序”选项卡，然后在底部抽屉中单击“添加”。
-	根据提示创建一个新的“本机客户端应用程序”。
    -	应用程序的“名称”向最终用户描述你的应用程序
    -	“重定向 URI”是 Azure AD 要用来返回令牌响应的方案与字符串组合。输入一个值，例如 `http://DirectorySearcher`。
-	完成注册后，AAD 将为应用程序分配唯一的客户端标识符。在后面的部分中将会用到此值，因此，请从“配置”选项卡复制此值。
- 另外，请在“配置”选项卡中，找到“针对其他应用程序的权限”部分。对于“Azure Active Directory”应用程序，在“委托的权限”下添加“访问组织的目录”权限。这样，你的应用程序便可以在 Graph API 中查询用户。

## *2.安装并配置 ADAL*
将应用程序注册到 Azure AD 后，可以安装 ADAL 并编写标识相关的代码。为了使 ADAL 能够与 Azure AD 通信，需要为 ADAL 提供一些有关应用程序的注册信息。
-	首先，使用程序包管理器控制台将 ADAL 添加到解决方案中的各个项目。

`
PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -ProjectName DirectorySearcherLib
`

`
PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -ProjectName DirSearchClient-Android
`

`
PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -ProjectName DirSearchClient-Desktop
`

`
PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -ProjectName DirSearchClient-iOS
`

`
PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -ProjectName DirSearchClient-Universal
`

- 你应会发现，每个项目中添加了两个库 - ADAL 的 PCL 部分，和特定于平台的部分。

-	在 DirectorySearcherLib 项目中，打开 `DirectorySearcher.cs`。更改类成员值以反映你在 Azure 门户预览中输入的值。只要使用 ADAL，你的代码就会引用这些值。
    -	`tenant` 是 Azure AD 租户的域，例如 contoso.partner.onmschina.cn
    -	`clientId` 是从门户复制的应用程序 clientId。
    - `returnUri` 是你在门户中输入的 redirectUri，例如 `http://DirectorySearcher`。

## *3.使用 ADAL 从 Azure AD 获取令牌*
*几乎*所有的应用程序身份验证逻辑都在 `DirectorySearcher.SearchByAlias(...)` 中。在特定于平台的项目中，所要做的一切就是将上下文参数传递到 `DirectorySearcher` PCL。

- 首先打开 `DirectorySearcher.cs`，然后将一个新参数添加到 `SearchByAlias(...)` 方法。`IPlatformParameters` 是上下文参数，用于封装 ADAL 需要对其执行身份验证的特定于平台的对象。

	C#
		
		public static async Task<List<User>> SearchByAlias(string alias, IPlatformParameters parent)
		{
		

- 接下来，初始化 `AuthenticationContext`（ADAL 的主类）。你将在此处传递 ADAL 与 Azure AD 通信时所需的坐标。然后调用 `AcquireTokenAsync(...)`，该类将会接受 `IPlatformParameters` 对象，并调用所需的身份验证流来向应用程序返回令牌。

	C#

		...
		AuthenticationResult authResult = null;
		try
		{
			AuthenticationContext authContext = new AuthenticationContext(authority);
			authResult = await authContext.AcquireTokenAsync(graphResourceUri, clientId, returnUri, parent);
		}
		catch (Exception ee)
		{
			results.Add(new User { error = ee.Message });
			return results;
		}
		...

- `AcquireTokenAsync(...)` 首先会尝试返回请求资源（在本例中为 Graph API）的令牌，而不提示用户输入其凭据（通过缓存或刷新旧令牌）。仅在必要时，它才会在获取请求的令牌之前，向用户显示 Azure AD 登录页。


- 然后，你可以在 Authorization 标头中将访问令牌附加到 Graph API 请求：

	C#
		
		...
		    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", authResult.AccessToken);
		...


这就是需要针对 `DirectorySearcher` PCL 和应用程序标识相关代码执行的所有操作。余下的操作是在每个平台的视图中调用 `SearchByAlias(...)` 方法，并根据需要添加代码来正确处理 UI 生命周期。

####Android：
- 在 `MainActivity.cs` 中，在按钮单击处理程序中添加对 `SearchByAlias(...)` 的调用：

	C#

		List<User> results = await DirectorySearcher.SearchByAlias(searchTermText.Text, new PlatformParameters(this));

- 还需要重写 `OnActivityResult` 生命周期方法，以将任何身份验证重定向转发回到相应的方法。ADAL 在 Android 中为此提供了帮助器方法：

	C#
		
		...
		protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
		{
		    base.OnActivityResult(requestCode, resultCode, data);
		    AuthenticationAgentContinuationHelper.SetAuthenticationAgentContinuationEventArgs(requestCode, resultCode, data);
		}
		...


####Windows 桌面：
- 在 `MainWindow.xaml.cs` 中，只需调用 `SearchByAlias(...)`，并在桌面的 `PlatformParameters` 对象中传递 `WindowInteropHelper`：

	C#
		
		List<User> results = await DirectorySearcher.SearchByAlias(
		  SearchTermText.Text,
		  new PlatformParameters(PromptBehavior.Auto, this.Handle));


####iOS：
- 在 `DirSearchClient_iOSViewController.cs` 中，iOS `PlatformParameters` 对象只会引用视图控制器：

	C#
		
		List<User> results = await DirectorySearcher.SearchByAlias(
		  SearchTermText.Text,
		  new PlatformParameters(PromptBehavior.Auto, this.Handle));


####Windows 通用：
- 在 Windows 通用中，打开 `MainPage.xaml.cs` 并实现 `Search` 方法，该方法会根据需要，使用共享项目中的帮助器方法来更新 UI。

	C#

		...
		    List<User> results = await DirectorySearcherLib.DirectorySearcher.SearchByAlias(SearchTermText.Text, new PlatformParameters(PromptBehavior.Auto, false));
		...


祝贺你！ 现在，你已创建一个有效的 Xamarin 应用程序，它可以对用户进行身份验证，并使用 OAuth 2.0 在五个不同的平台上安全调用 Web API。如果你尚未这样做，可以在租户中填充一些用户。运行你的 DirectorySearcher 应用程序，并使用这些用户之一进行登录。根据用户的 UPN 搜索其他用户。

使用 ADAL 可以方便地将常见标识功能合并到应用中。它会负责所有的繁琐工作 - 缓存管理、OAuth 协议支持、向用户显示登录名 UI、刷新已过期的令牌，等等。你只需要真正了解一个 API 调用，即 `authContext.AcquireToken*(…)`。

[此处](https://github.com/AzureADQuickStarts/NativeClient-MultiTarget-DotNet/archive/complete.zip)提供了已完成示例（无需配置值）供你参考。现在，你可以转到其他标识方案。你可能想要尝试：

[使用 Azure AD 保护 .NET Web API >>](/documentation/articles/active-directory-devquickstarts-webapi-dotnet/)

[AZURE.INCLUDE [active-directory-devquickstarts-additional-resources](../../includes/active-directory-devquickstarts-additional-resources.md)]

<!---HONumber=Mooncake_Quality_Review_0104_2017-->
