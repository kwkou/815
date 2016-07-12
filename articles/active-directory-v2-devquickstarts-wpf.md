<properties
	pageTitle="Azure AD v2.0 .NET 本机应用 | Azure"
	description="如何构建一个使用个人 Microsoft 帐户和工作或学校帐户来登录用户的 .NET 本机应用。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="02/20/2016"
	wacn.date="06/27/2016"/>

# 将登录凭据添加到 Windows 桌面应用

v2.0 终结点可让你快速地将身份验证添加桌面应用，同时支持个人 Microsoft 帐户以及工作或学校帐户。它也可让你应用程序安全地与后端 Web API，以及 [Microsoft Graph](https://graph.microsoft.io) 和多个 [Office 365 统一 API](https://www.msdn.com/office/office365/howto/authenticate-Office-365-APIs-using-v2) 进行通信。

> [AZURE.NOTE]
	v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。

对于[在设备上运行的 .NET 本机应用](/documentation/articles/active-directory-v2-flows/#mobile-and-native-apps)，Azure AD 提供 Active Directory 身份验证库 (ADAL)。在本质上，ADAL 的唯一用途就是方便应用获取用于调用 Web 服务的令牌。为了演示这种简便性，我们生成了一个 .NET WPF 待办事项列表应用，其中包括：

-	使用 [OAuth 2.0 身份验证协议](/documentation/articles/active-directory-v2-protocols/#oauth2-authorization-code-flow)来登录用户和获取访问令牌。
-	安全调用受 OAuth 2.0 保护的后端待办事项列表 Web 服务。
-	将用户注销。

## 下载示例代码

本教程的代码[在 GitHub 上](https://github.com/AzureADQuickStarts/AppModelv2-NativeClient-DotNet)维护。若要遵照该代码，你可以[下载 .zip 格式应用骨架](https://github.com/AzureADQuickStarts/AppModelv2-NativeClient-DotNet/archive/skeleton.zip)，或克隆该骨架：

		git clone --branch skeleton https://github.com/AzureADQuickStarts/AppModelv2-NativeClient-DotNet.git

本教程末尾也提供完成的应用。

## 注册应用程序
在 [apps.dev.microsoft.com](https://apps.dev.microsoft.com) 中创建新的应用程序，或遵循以下[详细步骤](/documentation/articles/active-directory-v2-app-registration/)。请确保：

- 复制分配给应用程序的**应用程序 ID**，因为稍后将要用到。
- 为应用添加**移动**平台。
- 从门户复制**重定向 URI**。必须使用默认值 `urn:ietf:wg:oauth:2.0:oob`。

## 安装并配置 ADAL
现在有了已向 Microsoft 注册的应用，可以安装 ADAL 并编写与标识相关的代码。为了使 ADAL 能够与 v2.0 终结点通信，你需要提供一些与应用注册相关的信息。

-	首先，使用包管理器控制台将 ADAL 添加到 TodoListClient 项目。


		PM> Install-Package Microsoft.Experimental.IdentityModel.Clients.ActiveDirectory -ProjectName TodoListClient -IncludePrerelease


-	在 TodoListClient 项目中打开 `app.config`。替换 `<appSettings>` 节中的元素值，以反映你在应用注册门户中输入的值。只要使用 ADAL，你的代码就会引用这些值。
    -	`ida:ClientId` 是从门户复制的、应用的**应用程序 ID**。
    -	`ida:RedirectUri` 是来自门户的**重定向 URI**。
- 在 TodoList-Service 项目中，打开项目根目录中的 `web.config`。  
    - 将 `ida:Audience` 值替换为来自门户的相同**应用程序 ID**。

## 使用 ADAL 获取令牌
ADAL 遵守的基本原理是，每当应用程序需要访问令牌时，你只需调用 `authContext.AcquireToken(...)`，然后 ADAL 就会负责其余的工作。

-	在 `TodoListClient` 项目中，打开 `MainWindow.xaml.cs` 并找到 `OnInitialized(...)` 方法。第一步是初始化应用程序的 `AuthenticationContext`（ADAL 的主类）。你将在此处传递 ADAL 与 Azure AD 通信时所需的坐标，并告诉 ADAL 如何缓存令牌。

		C#
		protected override async void OnInitialized(EventArgs e)
		{
				base.OnInitialized(e);
		
				authContext = new AuthenticationContext(authority, new FileCache());
				AuthenticationResult result = null;
				...
		}


- 当应用程序启动时，我们希望检查并查看用户是否已登录应用。但是，我们不想在此时调用登录 UI，而是让用户单击“登录”才执行此操作。另外，请在 `OnInitialized(...)` 方法中：

		C#
		// As the app starts, we want to check to see if the user is already signed in.
		// You can do so by trying to get a token from ADAL, passing in the parameter
		// PromptBehavior.Never.  This forces ADAL to throw an exception if it cannot
		// get a token for the user without showing a UI.
		
		try
		{
				result = await authContext.AcquireTokenAsync(new string[] { clientId }, null, clientId, redirectUri, new PlatformParameters(PromptBehavior.Never, null));
		
				// If we got here, a valid token is in the cache.  Proceed to
				// fetch the user's tasks from the TodoListService via the
				// GetTodoList() method.
		
				SignInButton.Content = "Clear Cache";
				GetTodoList();
		}
		catch (AdalException ex)
		{
				if (ex.ErrorCode == "user_interaction_required")
				{
						// If user interaction is required, the app should take no action,
						// and simply show the user the sign in button.
				}
				else
				{
						// Here, we catch all other AdalExceptions
		
						string message = ex.Message;
						if (ex.InnerException != null)
						{
								message += "Inner Exception : " + ex.InnerException.Message;
						}
						MessageBox.Show(message);
				}
		}


- 如果用户未登录而按下“登录”按钮，我们希望调用登录 UI 并让用户输入其凭据。实现“登录”按钮处理程序：
		
		C#
		private async void SignIn(object sender = null, RoutedEventArgs args = null)
		{
				// TODO: Sign the user out if they clicked the "Clear Cache" button
		
				// If the user clicked the 'Sign-In' button, force
				// ADAL to prompt the user for credentials by specifying
				// PromptBehavior.Always.  ADAL will get a token for the
				// TodoListService and cache it for you.
		
				AuthenticationResult result = null;
				try
				{
						result = await authContext.AcquireTokenAsync(new string[] { clientId }, null, clientId, redirectUri, new PlatformParameters(PromptBehavior.Always, null));
						SignInButton.Content = "Clear Cache";
						GetTodoList();
				}
				catch (AdalException ex)
				{
						// If ADAL cannot get a token, it will throw an exception.
						// If the user canceled the login, it will result in the
						// error code 'authentication_canceled'.
		
						if (ex.ErrorCode == "authentication_canceled")
						{
								MessageBox.Show("Sign in was canceled by the user");
						}
						else
						{
								// An unexpected error occurred.
								string message = ex.Message;
								if (ex.InnerException != null)
								{
										message += "Inner Exception : " + ex.InnerException.Message;
								}
		
								MessageBox.Show(message);
						}
		
						return;
				}
		
		}


- 如果用户成功登录，ADAL 将为你接收和缓存令牌，然后你可以放心地继续调用 `GetTodoList()` 方法。获取用户任务的剩余步骤是实现 `GetTodoList()` 方法。

		C#
		private async void GetTodoList()
		{
				AuthenticationResult result = null;
				try
				{
						// Here, we try to get an access token to call the TodoListService
						// without invoking any UI prompt.  PromptBehavior.Never forces
						// ADAL to throw an exception if it cannot get a token silently.
		
						result = await authContext.AcquireTokenAsync(new string[] { clientId }, null, clientId, redirectUri, new PlatformParameters(PromptBehavior.Never, null));
				}
				catch (AdalException ex)
				{
						// ADAL couldn't get a token silently, so show the user a message
						// and let them click the Sign-In button.
		
						if (ex.ErrorCode == "user_interaction_required")
						{
								MessageBox.Show("Please sign in first");
								SignInButton.Content = "Sign In";
						}
						else
						{
								// In any other case, an unexpected error occurred.
		
								string message = ex.Message;
								if (ex.InnerException != null)
								{
										message += "Inner Exception : " + ex.InnerException.Message;
								}
								MessageBox.Show(message);
						}
		
						return;
				}
		
				// Once the token has been returned by ADAL,
		    // add it to the http authorization header,
		    // before making the call to access the To Do list service.
		
		    httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.Token);
		
				...



- When the user is done managing their To-Do List, they may finally sign out of the app by clicking the "Clear Cache" button.

		C#
		private async void SignIn(object sender = null, RoutedEventArgs args = null)
		{
				// If the user clicked the 'clear cache' button,
				// clear the ADAL token cache and show the user as signed out.
				// It's also necessary to clear the cookies from the browser
				// control so the next user has a chance to sign in.
		
				if (SignInButton.Content.ToString() == "Clear Cache")
				{
						TodoList.ItemsSource = string.Empty;
						authContext.TokenCache.Clear();
						ClearCookies();
						SignInButton.Content = "Sign In";
						return;
				}
		
				...


## 运行

祝贺你！ 现在，你已创建一个有效的 .NET WPF 应用，它可以对用户进行身份验证，使用 OAuth 2.0 安全调用 Web API。运行两个项目，并以个人的 Microsoft 或工作/学校的帐户登录。将任务添加到该用户的待办事项列表。注销，然后以其他用户的身份重新登录，以查看其他用户的待办事项列表。关闭应用程序，然后重新运行它。可以看到，用户的会话保持不变 - 这是因为应用在本地文件中缓存了令牌。

使用 ADAL 可以通过个人和工作帐户方便地将常用标识功能合并到应用。它会负责所有的繁琐工作 - 缓存管理、OAuth 协议支持、向用户显示登录名 UI、刷新已过期的令牌，等等。你只需要真正了解一个 API 调用，即 `authContext.AcquireTokenAsync(...)`。

[此处以 .zip 格式提供了](https://github.com/AzureADQuickStarts/AppModelv2-NativeClient-DotNet/archive/complete.zip)完整示例（不包括配置值），你也可以从 GitHub 克隆该示例：

		git clone --branch complete https://github.com/AzureADQuickStarts/AppModelv2-NativeClient-DotNet.git

## 后续步骤

现在，可以转到更高级的主题。你可能想要尝试：

- [使用 v2.0 终结点保护 TodoListService Web API >>](/documentation/articles/active-directory-v2-devquickstarts-dotnet-api/)

有关更多资源，请查看：

- [v2.0 开发人员指南 >>](/documentation/articles/active-directory-appmodel-v2-overview/)
- [堆栈溢出“adal”标记 >>](http://stackoverflow.com/questions/tagged/adal)

<!---HONumber=Mooncake_0516_2016-->