<properties
	pageTitle="Azure AD .NET 入门 | Azure"
	description="如何生成一个与 Azure AD 集成以进行身份验证和授权的 .NET MVC Web API。"
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
	ms.author="dastrock"
	wacn.date="01/09/2017"/>  



# 使用 Azure AD 提供的持有者令牌保护 Web API

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

如果你正在生成的应用程序会提供对受保护资源的访问，则你需要知道如何防止有人在未经授权的情况下访问这些资源。借助 Azure AD，你只需编写几行代码，就能使用 OAuth Bearer 2.0 访问令牌简单直接地保护 Web API。

在 Asp.NET Web Apps 中，你可以使用 .NET Framework 4.5 中包含的社区驱动 OWIN 中间件的 Microsoft 实现来达到此目的。现在，我们将使用 OWIN 来生成“待办事项列表”Web API：
-	指定要保护哪些 API。
-	验证 Web API 调用是否包含有效的访问令牌。

为此，你需要：

1. 将一个应用程序注册到 Azure AD
2. 将应用程序设置为使用 OWIN 身份验证管道。
3. 配置一个客户端应用程序用于调用待办事项列表 Web API

若要开始，请[下载应用程序框架](https://github.com/AzureADQuickStarts/WebAPI-Bearer-DotNet/archive/skeleton.zip)或[下载已完成的示例](https://github.com/AzureADQuickStarts/WebAPI-Bearer-DotNet/archive/complete.zip)。每个下载项目都是 Visual Studio 2013 解决方案。你还需要一个用于注册应用程序的 Azure AD 租户。如果你没有此租户，请[了解如何获取租户](/documentation/articles/active-directory-howto-tenant/)。


## *1.将一个应用程序注册到 Azure AD*
若要保护你的应用程序，首先需要在租户中创建一个应用程序，并为 Azure AD 提供一些关键信息。

-	登录到 [Azure 管理门户](https://manage.windowsazure.cn)
-	在左侧的导航栏中单击“Active Directory”
-	选择要在其中注册应用程序的租户。
-	单击“应用程序”选项卡，然后在底部抽屉中单击“添加”。
-	根据提示创建一个新的 **Web 应用程序和/或 WebAPI**。
    -	应用程序的“名称”向最终用户描述你的应用程序。输入“待办事项列表服务”。
    -	“重定向 URI”是 Azure AD 要用来返回应用程序请求的任何令牌的方案与字符串组合。为此值输入 `https://localhost:44321/`。
-	完成注册后，导航到“配置”选项卡并找到“应用 ID URI”字段。为此值输入特定于租户的标识符，例如 `https://contoso.partner.onmschina.cn/TodoListService`
- 保存配置。保持门户的打开状态 - 稍后你还需要注册客户端应用程序。

## *2.将应用程序设置为使用 OWIN 身份验证管道*

将应用程序注册到 Azure AD 后，需要将应用程序设置为与 Azure AD 通信，以验证传入请求和令牌。

-	若要开始，请打开解决方案，然后使用 Package Manager Console 将 OWIN 中间件 NuGet 包添加到 TodoListService 项目。


	PM> Install-Package Microsoft.Owin.Security.ActiveDirectory -ProjectName TodoListService
	PM> Install-Package Microsoft.Owin.Host.SystemWeb -ProjectName TodoListService


-	将名为 `Startup.cs` 的 OWIN 启动类添加到 TodoListService 项目。右键单击项目，选择“添加”-->“新建项”，然后搜索“OWIN”。当你的应用程序启动时，该 OWIN 中间件将调用 `Configuration(…)` 方法。
-	将类声明更改为 `public partial class Startup` - 我们已在另一个文件中实现了此类的一部分。在 `Configuration(…)` 方法中，调用 ConfgureAuth(...) 以设置 Web 应用的身份验证。

C#
		
	public partial class Startup
	{
	    public void Configuration(IAppBuilder app)
	    {
	        ConfigureAuth(app);
	    }
	}


-	打开文件 `App_Start\Startup.Auth.cs` 并实现 `ConfigureAuth(…)` 方法。在 `WindowsAzureActiveDirectoryBearerAuthenticationOptions` 中提供的参数将充当应用程序与 Azure AD 通信时使用的坐标。

C#
		
	public void ConfigureAuth(IAppBuilder app)
	{
	    app.UseWindowsAzureActiveDirectoryBearerAuthentication(
	        new WindowsAzureActiveDirectoryBearerAuthenticationOptions
	        {
	            Audience = ConfigurationManager.AppSettings["ida:Audience"],
	            Tenant = ConfigurationManager.AppSettings["ida:Tenant"]
	        });
	}


-	现在，你可以使用 `[Authorize]` 属性并结合 JWT 持有者身份验证来保护控制器和操作。使用 authorize 标记修饰 `Controllers\TodoListController.cs` 类。这会强制用户在访问该页面之前登录。

C#
		
	[Authorize]
	public class TodoListController : ApiController
	{


- 如果已授权的调用方成功调用了某个 `TodoListController` API，该操作可能需要访问有关调用方的信息。OWIN 通过 `ClaimsPrincpal` 对象提供对持有者令牌中的声明的访问。
- Web API 的一个常见要求是验证令牌中的“作用域”- 这可确保最终用户授予访问待办事项列表服务所需的权限：

C#
		
	public IEnumerable<TodoItem> Get()
	{
	    // user_impersonation is the default permission exposed by applications in AAD
	    if (ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/scope").Value != "user_impersonation")
	    {
	        throw new HttpResponseException(new HttpResponseMessage {
	          StatusCode = HttpStatusCode.Unauthorized,
	          ReasonPhrase = "The Scope claim does not contain 'user_impersonation' or scope claim not found"
	        });
	    }
	    ...
	}


-	最后，打开位于 TodoListService 项目根目录中的 `web.config` 文件，并在 `<appSettings>` 节中输入你的配置值。
  -	`ida:Tenant` 是 Azure AD 租户的名称，例如“contoso.partner.onmschina.cn”。
  -	`ida:Audience` 是你在 Azure 门户预览中为应用程序输入的应用程序 ID URI。

## *3.配置客户端应用程序并运行服务*
需要先配置待办事项列表客户端，使它能够从 AAD 获取令牌并可调用服务，然后，你才能看到待办事项服务的运行情况。

- 导航回到 [Azure 管理门户](https://manage.windowsazure.cn)
- 在 Azure AD 租户中创建新的应用程序，然后在最终提示中选择“本机客户端应用程序”。
    -	应用程序的“名称”向最终用户描述你的应用程序
    -	为“重定向 URI”值输入 `http://TodoListClient/`。
- 完成注册后，AAD 将为应用分配唯一的**客户端 ID**。在后面的步骤中将会用到此值，因此，请从“配置”选项卡复制此值。
- 另外，请在“配置”选项卡中，找到“针对其他应用程序的权限”部分。单击“添加应用程序”。 在“显示”下拉列表中选择“所有应用”，然后单击上方的复选标记。找到并单击你的待办事项列表服务，然后单击底部的复选标记以添加该应用程序。从“委托的权限”下拉列表中选择“访问待办事项列表服务”，然后保存配置。


- 在 Visual Studio 中，打开 TodoListClient 项目中的 `App.config`，然后在 `<appSettings>` 节中输入你的配置值。
  -	`ida:Tenant` 是 Azure AD 租户的名称，例如“contoso.partner.onmschina.cn”。
  -	`ida:ClientId` 是从 Azure 门户预览复制的。
  -	`todo:TodoListResourceId` 是你在 Azure 门户预览中为待办事项列表服务应用程序输入的应用程序 ID URI。

最后，清理、生成并运行每个项目！ 如果你尚未这样做，现在可以使用 *.partner.onmschina.cn 域在租户中创建一个新的用户。使用该用户登录到待办事项列表客户端，并向用户的待办事项列表添加一些任务。

[此处](https://github.com/AzureADQuickStarts/WebAPI-Bearer-DotNet/archive/complete.zip)提供了已完成示例（无需配置值）供你参考。现在，你可以转到其他标识方案。

[AZURE.INCLUDE [active-directory-devquickstarts-additional-resources](../../includes/active-directory-devquickstarts-additional-resources.md)]

<!---HONumber=Mooncake_Quality_Review_0104_2017-->
