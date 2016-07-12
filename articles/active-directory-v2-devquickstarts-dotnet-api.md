<properties
	pageTitle="Azure AD v2.0 .NET Web API | Azure"
	description="如何构建一个可从个人 Microsoft 帐户及公司或学校帐户接受令牌的 .NET MVC Web API。"
	services="active-directory"
	documentationCenter=".net"
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="02/20/2016"
	wacn.date="06/28/2016"/>

# 保护 MVC Web API

Azure Active Directory 的 v2.0 终结点可让你使用 [OAuth 2.0](/documentation/articles/active-directory-v2-protocols/#oauth2-authorization-code-flow) 访问令牌保护 Web API，具有个人 Microsoft 帐户以及公司或学校帐户的用户也能够安全访问 Web API。

> [AZURE.NOTE]
	v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。

在 ASP.NET Web API 中，你可以使用随附在 .NET Framework 4.5 中的 Microsoft OWIN 中间件来完成此操作。在此处，我们将使用 OWIN 构建可让客户端通过用户待办事项列表创建和读取任务的“待办事项列表”MVC Web API。Web API 将验证传入的请求是否包含有效的访问令牌，并拒绝受保护路由上未通过验证的所有请求。

## 下载
本教程的代码[在 GitHub 上](https://github.com/AzureADQuickStarts/AppModelv2-WebAPI-DotNet)维护。若要遵照该代码，你可以[下载 .zip 格式应用骨架](https://github.com/AzureADQuickStarts/AppModelv2-WebAPI-DotNet/archive/skeleton.zip)，或克隆该骨架：


		git clone --branch skeleton https://github.com/AzureADQuickStarts/AppModelv2-WebAPI-DotNet.git


该骨架应用包含简单 API 应用的重复使用代码，但是缺少与标识相关的所有部分。如果你不想要延用该应用，可以克隆或[下载完整的示例](https://github.com/AzureADQuickStarts/AppModelv2-WebAPI-DotNet/archive/skeleton.zip)。


		git clone https://github.com/AzureADQuickStarts/AppModelv2-WebAPI-DotNet.git


## 注册应用程序
在 [apps.dev.microsoft.com](https://apps.dev.microsoft.com) 中创建新的应用程序，或遵循以下[详细步骤](/documentation/articles/active-directory-v2-app-registration/)。请确保：

- 复制分配给应用程序的**应用程序 ID**，因为稍后将要用到。

此 Visual Studio 解决方案还包含“TodoListClient”，这是一个简单的 WPF 应用。TodoListClient 用于演示用户如何登录，以及如何向 Web API 发出请求。在本例中，TodoListClient 和 TodoListService 由同一个应用代表。若要配置 TodoListClient，你还应该：

- 为应用添加**移动**平台。
- 从门户复制**重定向 URI**。必须使用默认值 `urn:ietf:wg:oauth:2.0:oob`。


## 安装 OWIN

注册应用后，需要将应用设置为与 v2.0 终结点通信，以验证传入的请求和令牌。

- 若要开始，请打开解决方案，然后使用 Package Manager Console 将 OWIN 中间件 NuGet 包添加到 TodoListService 项目。


		PM> Install-Package Microsoft.Owin.Security.OAuth -ProjectName TodoListService
		PM> Install-Package Microsoft.Owin.Security.Jwt -ProjectName TodoListService
		PM> Install-Package Microsoft.Owin.Host.SystemWeb -ProjectName TodoListService


## 配置 OAuth 身份验证

- 将名为 `Startup.cs` 的 OWIN 启动类添加到 TodoListService 项目。右键单击项目，选择“添加”-->“新建项”，然后搜索“OWIN”。当你的应用程序启动时，该 OWIN 中间件将调用 `Configuration(…)` 方法。
- 将类声明更改为 `public partial class Startup` - 我们已在另一个文件中实现了此类的一部分。在 `Configuration(…)` 方法中，调用 ConfgureAuth(...) 以设置 Web 应用的身份验证。

		C#
		public partial class Startup
		{
		    public void Configuration(IAppBuilder app)
		    {
		        ConfigureAuth(app);
		    }
		}


- 打开文件 `App_Start\Startup.Auth.cs` 并实现 `ConfigureAuth(…)` 方法，以便将 Web API 设置为接受来自 v2.0 终结点的令牌。

		C#
		public void ConfigureAuth(IAppBuilder app)
		{
				var tvps = new TokenValidationParameters
				{
						// In this app, the TodoListClient and TodoListService
						// are represented using the same Application Id - we use
						// the Application Id to represent the audience, or the
						// intended recipient of tokens.
		
						ValidAudience = clientId,
		
						// In a real applicaiton, you might use issuer validation to
						// verify that the user's organization (if applicable) has
						// signed up for the app.  Here, we'll just turn it off.
		
						ValidateIssuer = false,
				};
		
				// Set up the OWIN pipeline to use OAuth 2.0 Bearer authentication.
				// The options provided here tell the middleware about the type of tokens
				// that will be recieved, which are JWTs for the v2.0 endpoint.
		
				// NOTE: The usual WindowsAzureActiveDirectoryBearerAuthenticaitonMiddleware uses a
				// metadata endpoint which is not supported by the v2.0 endpoint.  Instead, this
				// OpenIdConenctCachingSecurityTokenProvider can be used to fetch & use the OpenIdConnect
				// metadata document.
		
				app.UseOAuthBearerAuthentication(new OAuthBearerAuthenticationOptions
				{
						AccessTokenFormat = new Microsoft.Owin.Security.Jwt.JwtFormat(tvps, new OpenIdConnectCachingSecurityTokenProvider("https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration")),
				});
		}


- 现在，你可以使用 `[Authorize]` 属性并结合 OAuth 2.0 持有者身份验证来保护控制器和操作。使用 authorize 标记修饰 `Controllers\TodoListController.cs` 类。这会强制用户在访问该页面之前登录。

		C#
		[Authorize]
		public class TodoListController : ApiController
		{


- 如果已授权的调用方成功调用了某个 `TodoListController` API，该操作可能需要访问有关调用方的信息。OWIN 通过 `ClaimsPrincpal` 对象提供对持有者令牌中的声明的访问。  

		C#
		public IEnumerable<TodoItem> Get()
		{
		    // You can use the ClaimsPrincipal to access information about the
		    // user making the call.  In this case, we use the 'sub' or
		    // NameIdentifier claim to serve as a key for the tasks in the data store.
		
		    Claim subject = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier);
		
		    return from todo in todoBag
		           where todo.Owner == subject.Value
		           select todo;
		}


-	最后，打开位于 TodoListService 项目根目录中的 `web.config` 文件，并在 `<appSettings>` 节中输入你的配置值。
  -	`ida:Audience` 是你在门户中为应用输入的**应用程序 ID**。

## 配置客户端应用
需要先配置待办事项列表客户端，使它能够从 v2.0 终结点获取令牌并可调用服务，然后，你才能看到待办事项服务的运行情况。

- 在 TodoListClient 项目中打开 `App.config`，然后在 `<appSettings>` 节中输入你的配置值。
  -	从门户复制的 `ida:ClientId` 应用程序 ID。
	- `ida:RedirectUri` 是来自门户的**重定向 URI**。

最后，清理、生成并运行每个项目！ 现在，你已构建了一个可从个人 Microsoft 帐户及公司或学校帐户接受令牌的 .NET MVC Web API。请登录到 TodoListClient，然后调用该 Web API 以将任务添加到用户的待办事项列表。

[此处以 .zip 格式提供了](https://github.com/AzureADQuickStarts/AppModelv2-WebAPI-DotNet/archive/complete.zip)完整示例（不包括配置值），你也可以从 GitHub 克隆该示例：

		git clone --branch complete https://github.com/AzureADQuickStarts/AppModelv2-WebAPI-DotNet.git

## 后续步骤
现在，你可以转到其他主题。你可能想要尝试：

[从 Web 应用调用 Web API >>](/documentation/articles/active-directory-v2-devquickstarts-webapp-webapi-dotnet/)

有关更多资源，请查看：

- [v2.0 开发人员指南 >>](/documentation/articles/active-directory-appmodel-v2-overview/)

- [堆栈溢出“azure-active-directory”标记 >>](http://stackoverflow.com/questions/tagged/azure-active-directory)

<!---HONumber=Mooncake_0516_2016-->