<properties
    pageTitle="Azure AD .NET Web 应用入门 | Azure"
    description="生成一个与 Azure AD 集成以方便登录的 .NET MVC Web 应用。"
    services="active-directory"
    documentationcenter=".net"
    author="dstrockis"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="e15a41a4-dc5d-4c90-b3fe-5dc33b9a1e96"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="01/23/2017"
    wacn.date="03/13/2017"
    ms.author="dastrock" />  


# 使用 Azure AD 执行 ASP.NET Web 应用登录和注销
[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

使用 Azure Active Directory (Azure AD)，只需通过编写几行代码来提供单一登录和注销，就能简单外包 Web 应用的标识管理。通过使用适用于 .NET 的开放 Web 界面 (OWIN) 中间件的 Microsoft 实现，可以将用户登录到 ASP.NET Web 应用或从 ASP.NET Web 应用注销。NET Framework 4.5 中包含社区驱动 OWIN 中间件。本文演示如何使用 OWIN 执行以下操作：

- 使用 Azure AD 作为标识提供者将用户登录到 Web 应用。
- 显示某些用户信息。
- 将用户从应用中注销。

## 准备工作
- 下载[应用框架](https://github.com/AzureADQuickStarts/WebApp-OpenIdConnect-DotNet/archive/skeleton.zip)或下载[已完成的示例](https://github.com/AzureADQuickStarts/WebApp-OpenIdConnect-DotNet/archive/complete.zip)。
- 还需要一个用于注册应用的 Azure AD 租户。如果还没有 Azure AD 租户，请[了解如何获取租户](/documentation/articles/active-directory-howto-tenant/)。

准备好后，请按照以下 4 个部分中的步骤操作。

## 步骤 1：向 Azure AD 注册新应用
若要设置应用以便对用户进行身份验证，请先通过执行以下操作在租户中对其进行注册：

- 登录到 Azure 管理门户。
- 在左侧的导航栏中单击“Active Directory”。
- 选择你要在其中注册应用程序的租户。
- 单击“应用程序”选项卡，然后在底部抽屉中单击“添加”。
- 根据提示创建一个新的 **Web 应用程序和/或 WebAPI**。
    - 应用程序的**名称**向最终用户描述你的应用程序
    -	“登录 URL”是应用的基本 URL。框架的默认值为 `https://localhost:44320/`。
    - “应用 ID URI”是应用程序的唯一标识符。约定是使用 `https://<tenant-domain>/<app-name>`，例如 `https://contoso.partner.onmschina.cn/my-first-aad-app`
- 完成注册后，AAD 将为应用分配唯一的客户端标识符。在后面的部分中将会用到此值，因此，请从“配置”选项卡复制此值。

## 步骤 2：将应用设置为使用 OWIN 身份验证管道
在此步骤中，将 OWIN 中间件配置为使用 OpenID Connect 身份验证协议。使用 OWIN 发出登录和注销请求、管理用户会话以及获取用户信息等等。

1. 若要开始，请使用包管理器控制台将 OWIN 中间件 NuGet 包添加到项目。


		PM> Install-Package Microsoft.Owin.Security.OpenIdConnect
		PM> Install-Package Microsoft.Owin.Security.Cookies
		PM> Install-Package Microsoft.Owin.Host.SystemWeb


2. 若要将称为 `Startup.cs` 的 OWIN 启动类添加到项目，请右键单击项目，依次选择“添加”、“新建项”，然后搜索“OWIN”。该应用启动时，OWIN 中间件会调用 **Configuration(...)** 方法。
3. 将类声明更改为 `public partial class Startup`。我们已在另一个文件中实现了此类的一部分。在 **Configuration(...)** 方法中，调用 **ConfgureAuth(...)** 以设置应用的身份验证。

	C#

		public partial class Startup
		{
			public void Configuration(IAppBuilder app)
			{
				ConfigureAuth(app);
			}
		}


4. 打开 App\_Start\\Startup.Auth.cs 文件，然后实现 **ConfigureAuth(...)** 方法。在 *OpenIDConnectAuthenticationOptions* 中提供的参数充当应用与 Azure AD 通信时使用的坐标。还需要设置 Cookie 身份验证，因为 OpenID Connect 中间件将在后台使用 Cookie。

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


5. 打开位于项目根目录中的 web.config 文件，然后在 `<appSettings>` 节中输入配置值。
  - `ida:ClientId`：在“步骤 1：向 Azure AD 注册新的应用”中从 Azure 门户复制的 GUID。
  - `ida:Tenant`：Azure AD 租户的名称（例如，contoso.partner.onmschina.cn）。
  - `ida:PostLogoutRedirectUri`：在成功完成注销请求后，告知 Azure AD 要将用户重定向到哪个位置的指示器。

## 步骤 3：使用 OWIN 向 Azure AD 发出登录和注销请求
现在，应用已正确配置为使用 OpenID Connect 身份验证协议来与 Azure AD 进行通信。OWIN 已处理有关创建身份验证消息、验证 Azure AD 提供的令牌以及保留用户会话的细节。你要做的就是为用户提供登录和注销方式。

1. 可以在控制器中使用授权标记，要求用户在访问特定页面之前登录。为此，请打开 Controllers\\HomeController.cs，然后将 `[Authorize]` 标记添加到 About 控制器。

	C#

		[Authorize]
		public ActionResult About()
		{
		  ...


2. 还可以使用 OWIN 直接从代码内部发出身份验证请求。为此，请打开 Controllers\\AccountController.cs。然后，在 SignIn() 和 SignOut() 操作中，发出 OpenID Connect 质询和注销请求。

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


3. 打开 Views\\Shared\_LoginPartial.cshtml 以向用户显示应用登录和注销链接，并在视图中打印用户名称。

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


## 步骤 4：显示用户信息
使用 OpenID Connect 对用户进行身份验证时，Azure AD 向应用返回 id\_token，其中包含有关用户的“声明”或断言。可以通过执行以下操作，使用这些声明对应用进行个性化设置：

1. 打开 Controllers\\HomeController.cs 文件。可以通过 `ClaimsPrincipal.Current` 安全主体对象访问控制器中的用户声明。

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

2. 构建并运行应用程序。如果尚未使用 partner.onmschina.cn 域在租户中创建一个新的用户，则现在可以执行此操作。方法如下：

  a.以该用户的身份登录，然后注意该用户的标识在顶部栏中的显示方式。

  b.注销，然后以租户中其他用户的身份重新登录。

  c.如果有浓厚的兴趣，可以注册并运行此应用的另一个实例（使用其自身的 clientId），然后观察单一登录的运作方式。

## 后续步骤
有关参考，请参阅[已完成的示例](https://github.com/AzureADQuickStarts/WebApp-OpenIdConnect-DotNet/archive/complete.zip)（无需配置值）。

现在，可以转到更高级的主题。例如，请尝试[使用 Azure AD 保护 Web API](/documentation/articles/active-directory-devquickstarts-webapi-dotnet/)。

[AZURE.INCLUDE [active-directory-devquickstarts-additional-resources](../../includes/active-directory-devquickstarts-additional-resources.md)]

<!---HONumber=Mooncake_0306_2017-->
<!---Update_Description: wording update -->