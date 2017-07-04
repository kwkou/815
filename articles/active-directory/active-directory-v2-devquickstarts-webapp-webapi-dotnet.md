<properties
    pageTitle="Azure AD v2.0 .NET Web 应用调用 API 入门 | Azure"
    description="如何构建一个 .NET MVC Web 应用，以通过用于登录的 Microsoft 个人帐户和工作或学校帐户调用 Web 服务。"
    services="active-directory"
    documentationcenter=".net"
    author="dstrockis"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="56be906e-71de-469d-9a5c-9fc08aae4223"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="01/23/2017"
    wacn.date="03/13/2017"
    ms.author="dastrock" />  


# 从 .NET Web 应用调用 API
通过 v2.0 终结点，可以快速地将身份验证添加 Web 应用和 Web API，同时支持个人 Microsoft 帐户以及工作或学校帐户。此外，我们将构建一个借助 Microsoft OWIN 中间件使用 OpenID Connect 将用户登录的 MVC Web 应用。该 Web 应用程序将获取受 OAuth 2.0 保护的 Web API 的 OAuth 2.0 访问令牌，用于创建、读取和删除给定用户的“待办事项列表”。

本教程着重介绍如何通过 MSAL 获取和使用 Web 应用中的访问令牌，在[此处](/documentation/articles/active-directory-v2-flows/#web-apps/)可找到完整介绍。可能必须先了解如何[将基本登录添加到 Web 应用](/documentation/articles/active-directory-v2-devquickstarts-dotnet-web/)，或者如何[正确保护 Web API](/documentation/articles/active-directory-v2-devquickstarts-dotnet-api/)。

> [AZURE.NOTE]
v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。

## 下载示例代码
本教程的代码[在 GitHub 上](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-WebAPI-OpenIdConnect-DotNet)维护。若要遵照该代码，你可以[下载 .zip 格式应用骨架](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-WebAPI-OpenIdConnect-DotNet/archive/skeleton.zip)，或克隆该骨架：

	git clone --branch skeleton https://github.com/AzureADQuickStarts/AppModelv2-WebApp-WebAPI-OpenIdConnect-DotNet.git

或者，可以[将已完成的应用以 .zip 格式下载](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-WebAPI-OpenIdConnect-DotNet/archive/complete.zip)，或克隆已完成的应用：

	git clone --branch complete https://github.com/AzureADQuickStarts/AppModelv2-WebApp-WebAPI-OpenIdConnect-DotNet.git

## 注册应用程序
在 [apps.dev.microsoft.com](https://apps.dev.microsoft.com/?referrer=/documentation/articles&deeplink=/appList) 中创建新的应用程序，或遵循以下[详细步骤](/documentation/articles/active-directory-v2-app-registration/)。请确保：

- 复制分配给应用程序的**应用程序 ID**，因为稍后将要用到。
- 创建**密码**类型的**应用密码**，并复制其值以备后用
- 为应用程序添加 **Web** 平台。
- 输入正确的**重定向 URI**。重定向 URI 向 Azure AD 指示身份验证响应应定向到的位置，本教程的默认值为 `https://localhost:44326/`。


## 安装 OWIN
使用包管理器控制台将 OWIN 中间件 NuGet 包添加到 `TodoList-WebApp` 项目。OWIN 中间件将用于发出登录和注销请求、管理用户的会话、获取有关用户的信息，等等。

	
	PM> Install-Package Microsoft.Owin.Security.OpenIdConnect -ProjectName TodoList-WebApp
	PM> Install-Package Microsoft.Owin.Security.Cookies -ProjectName TodoList-WebApp
	PM> Install-Package Microsoft.Owin.Host.SystemWeb -ProjectName TodoList-WebApp


## 登录用户
现在，我们要将 OWIN 中间件配置为使用 [OpenID Connect 身份验证协议](/documentation/articles/active-directory-v2-protocols/)。

- 打开位于 `TodoList-WebApp` 项目根目录中的 `web.config` 文件，然后在 `<appSettings>` 节中输入应用的配置值。
  - `ida:ClientId` 是在注册门户中为应用分配的**应用程序 ID**。
  - `ida:ClientSecret` 是在注册门户中创建的**应用密码**。
  - `ida:RedirectUri` 是在门户中输入的**重定向 URI**。
- 打开位于 `TodoList-Service` 项目根目录中的 `web.config` 文件，然后将 `ida:Audience` 替换为上述相同的**应用程序 ID**。
- 打开文件 `App_Start\Startup.Auth.cs`，为上述库添加 `using` 语句。
- 在同一个文件中，实现 `ConfigureAuth(...)` 方法。在 `OpenIDConnectAuthenticationOptions` 中提供的参数将充当应用程序与 Azure AD 通信时使用的坐标。

C#

	public void ConfigureAuth(IAppBuilder app)
	{
	    app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);
	
	    app.UseCookieAuthentication(new CookieAuthenticationOptions());
	
	    app.UseOpenIdConnectAuthentication(
	        new OpenIdConnectAuthenticationOptions
	        {

	                    // The `Authority` represents the v2.0 endpoint - https://login.microsoftonline.com/common/v2.0
	                    // The `Scope` describes the permissions that your app will need.  See https://azure.cn/documentation/articles/active-directory-v2-scopes/
	                    // In a real application you could use issuer validation for additional checks, like making sure the user's organization has signed up for your app, for instance.

	                    ClientId = clientId,
	                    Authority = String.Format(CultureInfo.InvariantCulture, aadInstance, "common", "/v2.0 "),
	                    Scope = "openid email profile offline_access",
	                    RedirectUri = redirectUri,
	                    PostLogoutRedirectUri = redirectUri,
	                    TokenValidationParameters = new TokenValidationParameters
	                    {
	                        ValidateIssuer = false,
	                    },

	                    // The `AuthorizationCodeReceived` notification is used to capture and redeem the authorization_code that the v2.0 endpoint returns to your app.

	                    Notifications = new OpenIdConnectAuthenticationNotifications
	                    {
	                        AuthenticationFailed = OnAuthenticationFailed,
	                        AuthorizationCodeReceived = OnAuthorizationCodeReceived,
	                    }

	        });
	}
	// ...


## 使用 MSAL 获取访问令牌
在 `AuthorizationCodeReceived` 通知中，我们想要使用[与 OpenID Connect 串联的 OAuth 2.0](/documentation/articles/active-directory-v2-protocols/)，以兑换待办事项列表服务的访问令牌的 authorization\_code。MSAL 可以简化此过程：

- 首先，安装 MSAL 预览版：

		PM> Install-Package Microsoft.Identity.Client -ProjectName TodoList-WebApp -IncludePrerelease

- 在 `App_Start\Startup.Auth.cs` 文件中为 MSAL 添加另一个 `using` 语句。
- 现在，添加一个新方法，即 `OnAuthorizationCodeReceived` 事件处理程序。此处理程序将使用 MSAL 获取待办事项列表 API 的访问令牌，并将 MSAL 令牌缓存中的令牌存储起来以供稍后使用：

	C#
	
		private async Task OnAuthorizationCodeReceived(AuthorizationCodeReceivedNotification notification)
		{
		        string userObjectId = notification.AuthenticationTicket.Identity.FindFirst("http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier").Value;
		        string tenantID = notification.AuthenticationTicket.Identity.FindFirst("http://schemas.microsoft.com/identity/claims/tenantid").Value;
		        string authority = String.Format(CultureInfo.InvariantCulture, aadInstance, tenantID, string.Empty);
		        ClientCredential cred = new ClientCredential(clientId, clientSecret);
	
		        // Here you ask for a token using the web app's clientId as the scope, since the web app and service share the same clientId.
		        app = new ConfidentialClientApplication(Startup.clientId, redirectUri, cred, new NaiveSessionCache(userObjectId, notification.OwinContext.Environment["System.Web.HttpContextBase"] as HttpContextBase)) {};
		        var authResult = await app.AcquireTokenByAuthorizationCodeAsync(new string[] { clientId }, notification.Code);
		}
		// ...


- 在 Web 应用中，MSAL 具有可扩展的令牌缓存，可用于存储令牌。此示例实现使用 http 会话存储来缓存令牌的 `NaiveSessionCache`。

<!-- TODO: Token Cache article -->



## 调用 Web API
现在可以实际使用在步骤 3 中获取的 access\_token。打开 Web 应用的 `Controllers\TodoListController.cs` 文件，此文件可向待办事项列表 API 发出所有 CRUD 请求。

- 此处可再次使用 MSAL，从 MSAL 缓存检索 access\_tokens。首先，将 MSAL 的 `using` 语句添加到此文件。
  
    `using Microsoft.Identity.Client;`  


- 在 `Index` 操作中，使用 MSAL 的 `AcquireTokenSilentAsync` 方法获取可用于读取待办事项列表服务数据的 access\_token：

	C#
	
		// ...
		string userObjectID = ClaimsPrincipal.Current.FindFirst("http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier").Value;
		string tenantID = ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/tenantid").Value;
		string authority = String.Format(CultureInfo.InvariantCulture, Startup.aadInstance, tenantID, string.Empty);
		ClientCredential credential = new ClientCredential(Startup.clientId, Startup.clientSecret);
		
		// Here you ask for a token using the web app's clientId as the scope, since the web app and service share the same clientId.
		app = new ConfidentialClientApplication(Startup.clientId, redirectUri, credential, new NaiveSessionCache(userObjectID, this.HttpContext)){};
		result = await app.AcquireTokenSilentAsync(new string[] { Startup.clientId });
		// ...
	

- 此示例接下来将生成的令牌添加到 HTTP GET 请求作为 `Authorization` 标头，让待办事项列表服务用于身份验证请求。
- 如果待办事项列表服务返回 `401 Unauthorized` 响应，则表示 MSAL 的 access\_tokens 由于某种原因而失效。在此情况下，你应该删除所有来自 MSAL 缓存的 access\_tokens，并向用户显示消息告知可能需要再次登录，而这会重新启动令牌获取流。

	C#
	
		// ...
		// If the call failed with access denied, then drop the current access token from the cache,
		// and show the user an error indicating they might need to sign-in again.
		if (response.StatusCode == System.Net.HttpStatusCode.Unauthorized)
		{
		        app.AppTokenCache.Clear(Startup.clientId);
	
		        return new RedirectResult("/Error?message=Error: " + response.ReasonPhrase + " You might need to sign in again.");
		}
		// ...


- 同样，如果 MSAL 由于任何原因而无法返回 access\_token，则应再次指示用户登录。这就像获取任何 `MSALException` 一样简单：

	C#
	
		// ...
		catch (MsalException ee)
		{
		        // If MSAL could not get a token silently, show the user an error indicating they might need to sign in again.
		        return new RedirectResult("/Error?message=An Error Occurred Reading To Do List: " + ee.Message + " You might need to log out and log back in.");
		}
		// ...
	

- 完全一样的 `AcquireTokenSilentAsync` 调用在 `Create` 和 `Delete` 操作中实现。在 Web 应用中，只要应用程序有需要，就可以使用此 MSAL 方法获取 access\_tokens。MSAL 将为用户获取、缓存和刷新令牌。

最后，生成并运行应用程序！ 使用 Microsoft 帐户或 Azure AD 帐户登录，随后你会看到该用户的标识在顶部导航栏中的显示方式。在用户的“待办事项列表”中添加和删除一些项，以查看 OAuth 2.0 保护的 API 调用的运行情况。Web 应用和 Web API 现在都使用行业标准的协议进行保护，你可以使用个人和工作/学校帐户来验证用户。

[此处](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-WebAPI-OpenIdConnect-DotNet/archive/complete.zip)提供了已完成示例（无需配置值）供你参考。

## 后续步骤
有关更多资源，请查看：

- [v2.0 开发人员指南 >>](/documentation/articles/active-directory-appmodel-v2-overview/)
- [堆栈溢出“azure-active-directory”标记 >>](http://stackoverflow.com/questions/tagged/azure-active-directory)

## 获取关于我们产品的安全更新
建议发生安全事件时获取相关通知，方法是访问[此页](https://technet.microsoft.com/security/dd252948)并订阅“安全公告通知”。

<!---HONumber=Mooncake_0306_2017-->
<!---Update_Description: wording update -->