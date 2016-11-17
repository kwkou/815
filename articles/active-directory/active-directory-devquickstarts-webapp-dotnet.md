<properties
	pageTitle="Azure AD .NET 入门 | Azure"
	description="如何生成一个与 Azure AD 集成以支持登录的 .NET MVC Web 应用。"
	services="active-directory"
	documentationCenter=".net"
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="09/16/2016"
	wacn.date="10/17/2016"
	ms.author="dastrock"/>

# 使用 Azure AD 执行 Web 应用登录和注销

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

使用 Azure AD，只需编写几行代码，就能简单直接地外包 Web 应用的标识管理，提供单一登录和注销。在 Asp.NET Web Apps中，你可以使用 .NET Framework 4.5 中包含的社区驱动 OWIN 中间件的 Microsoft 实现来达到此目的。现在，我们将使用 OWIN 来执行下列操作：
-	使用 Azure AD 作为标识提供者将用户登录到应用。
-	显示有关用户的一些信息。
-	从应用中注销用户。

为此，你需要：

1. 将一个应用程序注册到 Azure AD
2. 将应用程序设置为使用 OWIN 身份验证管道。
3. 使用 OWIN 向 Azure AD 发出登录和注销请求。
4. 列显有关用户的数据。

若要开始，请[下载应用程序框架](https://github.com/AzureADQuickStarts/WebApp-OpenIdConnect-DotNet/archive/skeleton.zip)或[下载已完成的示例](https://github.com/AzureADQuickStarts/WebApp-OpenIdConnect-DotNet/archive/complete.zip)。你还需要一个用于注册应用程序的 Azure AD 租户。如果你没有此租户，请[了解如何获取租户](/documentation/articles/active-directory-howto-tenant/)。

## *1.将一个应用程序注册到 Azure AD*
若要使应用程序对用户进行身份验证，你首先需要在租户中注册新的应用程序。

- 登录到 Azure 管理门户。
- 在左侧的导航栏中单击“Active Directory”。
- 选择你要在其中注册应用程序的租户。
- 单击“应用程序”选项卡，然后在底部抽屉中单击“添加”。
- 根据提示创建一个新的 **Web 应用程序和/或 WebAPI**。
    - 应用程序的**名称**向最终用户描述你的应用程序
    -	“登录 URL”是应用程序的基本 URL。框架的默认值为 `https://localhost:44320/`。
    - “应用程序 ID URI”是应用程序的唯一标识符。约定是使用 `https://<tenant-domain>/<app-name>`，例如 `https://contoso.partner.onmschina.cn/my-first-aad-app`
- 完成注册后，AAD 将为应用程序分配唯一的客户端标识符。在后面的部分中将会用到此值，因此，请从“配置”选项卡复制此值。

## *2.将应用程序设置为使用 OWIN 身份验证管道*
在这里，我们要将 OWIN 中间件配置为使用 OpenID Connect 身份验证协议。OWIN 将用于发出登录和注销请求、管理用户的会话、获取有关用户的信息，等等。

-	若要开始，请使用 Package Manager Console 将 OWIN 中间件 NuGet 包添加到项目。

		
	PM> Install-Package Microsoft.Owin.Security.OpenIdConnect
	PM> Install-Package Microsoft.Owin.Security.Cookies
	PM> Install-Package Microsoft.Owin.Host.SystemWeb


-	将称为 `Startup.cs` 的 OWIN 启动类添加到项目。右键单击项目，选择“添加”-->“新建项”，然后搜索“OWIN”。当你的应用程序启动时，该 OWIN 中间件将调用 `Configuration(...)` 方法。
-	将类声明更改为 `public partial class Startup` - 我们已在另一个文件中实现了此类的一部分。在 `Configuration(...)` 方法中，调用 ConfgureAuth(...) 以设置 Web 应用的身份验证

C#

	public partial class Startup
	{
	    public void Configuration(IAppBuilder app)
	    {
	        ConfigureAuth(app);
	    }
	}


-	打开文件 `App_Start\Startup.Auth.cs` 并实现 `ConfigureAuth(...)` 方法。在 `OpenIDConnectAuthenticationOptions` 中提供的参数将充当应用程序与 Azure AD 通信时使用的坐标。你还需要设置 Cookie 身份验证 - OpenID Connect 中间件将在幕后使用 Cookie。

C#

	public void ConfigureAuth(IAppBuilder app)
	{
	    app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

	    app.UseCookieAuthentication(new CookieAuthenticationOptions());

	    app.UseOpenIdConnectAuthentication(
	        new OpenIdConnectAuthenticationOptions
	        {
	            ClientId = clientId,
	            Authority = authority,
	            PostLogoutRedirectUri = postLogoutRedirectUri,
	        });
	}


-	最后，打开位于项目根目录中的 `web.config` 文件，并在 `<appSettings>` 节中输入你的配置值。
    -	应用程序的 `ida:ClientId` 是你在执行步骤 1 时从 Azure 门户复制的 Guid。
    -	`ida:Tenant` 是 Azure AD 租户的名称，例如“contoso.partner.onmschina.cn”。
    -	`ida:PostLogoutRedirectUri` 向 Azure AD 指示在成功完成注销请求后，要将用户重定向到哪个位置。

## *3.使用 OWIN 向 Azure AD 发出登录和注销请求*
现在，应用程序已正确配置为使用 OpenID Connect 身份验证协议来与 Azure AD 进行通信。OWIN 会代你处理有关创建身份验证消息、验证 Azure AD 提供的令牌以及保留用户会话的繁琐细节。你要做的一切就是提供某种方式让用户登录和注销。

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
	public void SignOut()
	{
	    // Send an OpenID Connect sign-out request.
	    HttpContext.GetOwinContext().Authentication.SignOut(
	        OpenIdConnectAuthenticationDefaults.AuthenticationType, CookieAuthenticationDefaults.AuthenticationType);
	}


-	现在，请打开 `Views\Shared\_LoginPartial.cshtml`。你将在其中向用户显示应用程序的登录和注销链接，用户名将在视图中列显。

HTML

	@if (Request.IsAuthenticated)
	{
	    <text>
	        <ul class="nav navbar-nav navbar-right">
	            <li class="navbar-text">
	                Hello, @User.Identity.Name!
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


## *4.显示用户信息*
使用 OpenID Connect 对用户进行身份验证时，Azure AD 将向应用程序返回 id\_token，其中包含有关用户的“声明”或断言。你可以使用这些声明来个性化应用程序：

- 打开 `Controllers\HomeController.cs` 文件。可以通过 `ClaimsPrincipal.Current` 安全主体对象访问控制器中的用户声明。

C#

	public ActionResult About()
	{
	    ViewBag.Name = ClaimsPrincipal.Current.FindFirst(ClaimTypes.Name).Value;
	    ViewBag.ObjectId = ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier").Value;
	    ViewBag.GivenName = ClaimsPrincipal.Current.FindFirst(ClaimTypes.GivenName).Value;
	    ViewBag.Surname = ClaimsPrincipal.Current.FindFirst(ClaimTypes.Surname).Value;
	    ViewBag.UPN = ClaimsPrincipal.Current.FindFirst(ClaimTypes.Upn).Value;
	
	    return View();
	}


最后，生成并运行应用程序！ 如果你尚未这样做，现在可以使用 *.partner.onmschina.cn 域在租户中创建一个新的用户。以该用户的身份登录，随后你会看到该用户的标识在顶部导航栏中的显示方式。注销，然后以租户中其他用户的身份重新登录。如果你有浓厚的兴趣，可以注册并运行此应用程序的另一个实例（使用其自身的 clientId），然后观察单一登录的运作方式。

[此处](https://github.com/AzureADQuickStarts/WebApp-OpenIdConnect-DotNet/archive/complete.zip)提供了已完成示例（无需配置值）供你参考。

现在，可以转到更高级的主题。你可能想要尝试：

[使用 Azure AD 保护 Web API >>](/documentation/articles/active-directory-devquickstarts-webapi-dotnet/)

[AZURE.INCLUDE [active-directory-devquickstarts-additional-resources](../../includes/active-directory-devquickstarts-additional-resources.md)]
 

<!---HONumber=Mooncake_1010_2016-->
