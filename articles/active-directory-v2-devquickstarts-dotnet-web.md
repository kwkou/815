<properties
	pageTitle="Azure AD v2.0 .NET Web 应用 | Azure"
	description="如何构建一个使用个人 Microsoft 帐户和工作或学校帐户来登录用户的 .NET MVC Web 应用。"
	services="active-directory"
	documentationCenter=".net"
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="02/20/2016"
	wacn.date="06/28/2016"/>

# 将登录凭据添加到 .NET MVC Web 应用

通过 v2.0 终结点，可以快速地将身份验证添加 Web 应用，同时支持个人 Microsoft 帐户以及工作或学校帐户。在 ASP.NET Web 应用中，你可以使用随附在 .NET Framework 4.5 中的 Microsoft OWIN 中间件来完成此操作。

> [AZURE.NOTE]
	v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。

 此处，我们将构建一个可以使用 OWIN 来将用户登录、显示有关用户的某些信息以及将用户从应用中注销的 Web 应用。
 
 ## 下载。
本教程的代码[在 GitHub 上](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIdConnect-DotNet)维护。若要遵照该代码，你可以[下载 .zip 格式应用骨架](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIdConnect-DotNet/archive/skeleton.zip)，或克隆该骨架：

		git clone --branch skeleton https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIdConnect-DotNet.git

本教程末尾也提供完成的应用。

## 注册应用程序
在 [apps.dev.microsoft.com](https://apps.dev.microsoft.com) 中创建新的应用程序，或遵循以下[详细步骤](/documentation/articles/active-directory-v2-app-registration/)。请确保：

- 复制分配给应用程序的**应用程序 ID**，因为稍后将要用到。
- 为应用程序添加 **Web** 平台。
- 输入正确的**重定向 URI**。重定向 URI 向 Azure AD 指示身份验证响应应定向到的位置，本教程的默认值为 `https://localhost:44326/`。

## 安装并配置 OWIN 身份验证
在这里，我们要将 OWIN 中间件配置为使用 OpenID Connect 身份验证协议。OWIN 将用于发出登录和注销请求、管理用户的会话、获取有关用户的信息，等等。

-	首先，打开位于项目根目录中的 `web.config` 文件，并在 `<appSettings>` 节中输入应用程序的配置值。
    -	`ida:ClientId` 是在注册门户中为应用分配的**应用程序 ID**。
    -	`ida:RedirectUri` 是在门户中输入的**重定向 URI**。

-	接下来，使用包管理器控制台将 OWIN 中间件 NuGet 包添加到项目。


		PM> Install-Package Microsoft.Owin.Security.OpenIdConnect
		PM> Install-Package Microsoft.Owin.Security.Cookies
		PM> Install-Package Microsoft.Owin.Host.SystemWeb
		

-	将称为 `Startup.cs` 的 OWIN 启动类添加到项目。右键单击项目，选择“添加”-->“新建项”，然后搜索“OWIN”。当你的应用程序启动时，该 OWIN 中间件将调用 `Configuration(...)` 方法。
-	将类声明更改为 `public partial class Startup` - 我们已在另一个文件中实现了此类的一部分。在 `Configuration(...)` 方法中，调用 ConfigureAuth(...) 以设置 Web 应用的身份验证。  

		C#
		[assembly: OwinStartup(typeof(Startup))]
		
		namespace TodoList_WebApp
		{
			public partial class Startup
			{
				public void Configuration(IAppBuilder app)
				{
					ConfigureAuth(app);
				}
			}
		}

-	打开文件 `App_Start\Startup.Auth.cs` 并实现 `ConfigureAuth(...)` 方法。在 `OpenIdConnectAuthenticationOptions` 中提供的参数将充当应用程序与 Azure AD 通信时使用的坐标。你还需要设置 Cookie 身份验证 - OpenID Connect 中间件将在幕后使用 Cookie。

		C#
		public void ConfigureAuth(IAppBuilder app)
					 {
							 app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);
		
							 app.UseCookieAuthentication(new CookieAuthenticationOptions());
		
							 app.UseOpenIdConnectAuthentication(
									 new OpenIdConnectAuthenticationOptions
									 {
											 // The `Authority` represents the v2.0 endpoint - https://login.microsoftonline.com/common/v2.0 
											 // The `Scope` describes the permissions that your app will need.  See https://azure.microsoft.com/documentation/articles/active-directory-v2-scopes/
											 // In a real application you could use issuer validation for additional checks, like making sure the user's organization has signed up for your app, for instance.
		
											 ClientId = clientId,
											 Authority = String.Format(CultureInfo.InvariantCulture, aadInstance, "common", "/v2.0 "),
											 RedirectUri = redirectUri,
											 Scope = "openid email profile",
											 ResponseType = "id_token",
											 PostLogoutRedirectUri = redirectUri,
											 TokenValidationParameters = new TokenValidationParameters
											 {
													 ValidateIssuer = false,
											 },
											 Notifications = new OpenIdConnectAuthenticationNotifications
											 {
													 AuthenticationFailed = OnAuthenticationFailed,
											 }
									 });
					 }


## 发送身份验证请求
现在，应用程序已正确配置为使用 OpenID Connect 身份验证协议来与 v2.0 终结点通信。OWIN 会代你处理有关创建身份验证消息、验证 Azure AD 提供的令牌以及保留用户会话的繁琐细节。你要做的一切就是提供某种方式让用户登录和注销。

- 可以在控制器中使用授权标记，要求用户在访问特定页面之前登录。打开 `Controllers\HomeController.cs`，然后将 `[Authorize]` 标记添加到 About 控制器。

		C#
		[Authorize]
		public ActionResult About()
		{
		  ...


-	还可以使用 OWIN 直接从代码内部发出身份验证请求。打开 `Controllers\AccountController.cs`。在 SignIn() 和 SignOut() 操作中，分别发出 OpenID Connect 质询和注销请求。

		C#
		public void SignIn()
		{
		    // Send an OpenID Connect sign-in request.
		    if (!Request.IsAuthenticated)
		    {
		        HttpContext.GetOwinContext().Authentication.Challenge(new AuthenticationProperties { RedirectUri = "/" }, OpenIdConnectAuthenticationDefaults.AuthenticationType);
		    }
		}
		
		// BUGBUG: Ending a session with the v2.0 endpoint is not yet supported.  Here, we just end the session with the web app.  
		public void SignOut()
		{
		    // Send an OpenID Connect sign-out request.
		    HttpContext.GetOwinContext().Authentication.SignOut(CookieAuthenticationDefaults.AuthenticationType);
		    Response.Redirect("/");
		}


-	现在，请打开 `Views\Shared\_LoginPartial.cshtml`。你将在其中向用户显示应用程序的登录和注销链接，用户名将在视图中列显。

		HTML
		@if (Request.IsAuthenticated)
		{
		    <text>
		        <ul class="nav navbar-nav navbar-right">
		            <li class="navbar-text">
		
		                @*The 'preferred_username' claim can be used for showing the user's primary way of identifying themselves.*@
		
		                Hello, @(System.Security.Claims.ClaimsPrincipal.Current.FindFirst("preferred_username").Value)!
		            </li>
		            <li>
		                @Html.ActionLink("Sign out", "SignOut", "Account")
		            </li>
		        </ul>
		    </text>
		}
		else
		{
		    <ul class="nav navbar-nav navbar-right">
		        <li>@Html.ActionLink("Sign in", "SignIn", "Account", routeValues: null, htmlAttributes: new { id = "loginLink" })</li>
		    </ul>
		}


## 显示用户信息
使用 OpenID Connect 对用户进行身份验证时，v2.0 终结点将向应用返回 id\_token，其中包含有关用户的[声明](/documentation/articles/active-directory-v2-tokens/#id_tokens)或断言。你可以使用这些声明来个性化应用程序：

- 打开 `Controllers\HomeController.cs` 文件。可以通过 `ClaimsPrincipal.Current` 安全主体对象访问控制器中的用户声明。

		C#
		[Authorize]
		public ActionResult About()
		{
		    ViewBag.Name = ClaimsPrincipal.Current.FindFirst("name").Value;
		
		    // The object ID claim will only be emitted for work or school accounts at this time.
		    Claim oid = ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier");
		    ViewBag.ObjectId = oid == null ? string.Empty : oid.Value;
		
		    // The 'preferred_username' claim can be used for showing the user's primary way of identifying themselves
		    ViewBag.Username = ClaimsPrincipal.Current.FindFirst("preferred_username").Value;
		
		    // The subject or nameidentifier claim can be used to uniquely identify the user
		    ViewBag.Subject = ClaimsPrincipal.Current.FindFirst("http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier").Value;
		
		    return View();
		}


## 运行

最后，生成并运行应用程序！ 使用个人 Microsoft 帐户或者工作或学校帐户登录，随后你会看到该用户的标识已出现在顶部导航栏中。Web 应用现在使用行业标准的协议进行保护，你可以使用个人和工作/学校帐户来验证用户。

[此处以 .zip 格式提供了](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIdConnect-DotNet/archive/complete.zip)完整示例（不包括配置值），你也可以从 GitHub 克隆该示例：

		git clone --branch complete https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIdConnect-DotNet.git

## 后续步骤

现在，可以转到更高级的主题。你可能想要尝试：

[使用 v2.0 终结点保护 Web API >>](/documentation/articles/active-directory-devquickstarts-webapi-dotnet/)

有关更多资源，请查看：

- [v2.0 开发人员指南 >>](/documentation/articles/active-directory-appmodel-v2-overview/)

- [堆栈溢出“azure-active-directory”标记 >>](http://stackoverflow.com/questions/tagged/azure-active-directory)

<!---HONumber=Mooncake_0516_2016-->