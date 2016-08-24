<properties
	pageTitle="Azure AD AngularJS 入门 | Azure"
	description="如何生成一个与 Azure AD 集成以方便登录，并使用 OAuth 调用 Azure AD 保护 API 的 AngularJS 单页面应用程序。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="05/16/2016"
	wacn.date="08/22/2016"/>


# 如何使用 Azure AD 保护 AngularJS 单页面应用程序

[AZURE.INCLUDE [active-directory-devquickstarts-switcher](../../includes/active-directory-devquickstarts-switcher)]

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide)]

使用 Azure AD 可以简单直接地在单页面应用程序中添加登录、注销和安全 OAuth API 调用。它允许应用程序使用用户的 Active Directory 帐户对用户进行身份验证，并使用 Azure AD 保护的任何 Web API（例如 Office 365 API 或 Azure API）。

对于浏览器中运行的 javascript 应用程序，Azure AD 提供 Active Directory 身份验证库，即 adal.js。在本质上，Adal.js 的唯一用途就是方便应用程序获取访问令牌。为了演示这种简便性，我们生成了一个 AngularJS 待办事项列表应用程序，其中包括：

- 使用 Azure AD 作为标识提供程序将用户登录到应用程序。
- 显示有关用户的一些信息。
- 使用 AAD 提供的持有者令牌安全调用应用程序的待办事项 API。
- 从应用程序中注销用户。

若要生成完整的工作应用程序，你需要：

2. 将应用程序注册到 Azure AD。
3. 安装 ADAL 并配置 SPA。
5. 使用 ADAL 来保护 SPA 中的页面。

若要开始，请[下载应用程序框架](https://github.com/AzureADQuickStarts/SinglePageApp-AngularJS-DotNet/archive/skeleton.zip)或[下载已完成的示例](https://github.com/AzureADQuickStarts/SinglePageApp-AngularJS-DotNet/archive/complete.zip)。你还需要一个可在其中创建用户和注册应用程序的 Azure AD 租户。如果你还没有租户，请[了解如何获取租户](/documentation/articles/active-directory-howto-tenant/)。

## 1.注册 DirectorySearcher 应用程序
若要使应用程序对用户进行身份验证并获取令牌，首先需要在 Azure AD 租户中注册该应用程序：

-	登录到 [Azure 管理门户](https://manage.windowsazure.cn)
-	在左侧的导航栏中单击“Active Directory”
-	选择要在其中注册应用程序的租户。
-	单击“应用程序”选项卡，然后在底部抽屉中单击“添加”。
-	根据提示创建一个新的 **Web 应用程序和/或 WebAPI**。
    -	应用程序的“名称”向最终用户描述你的应用程序。
    -	“重定向 URI”是 AAD 要将令牌返回到的位置。本示例的默认位置是 `https://localhost:44326/`
-	完成注册后，AAD 将为应用程序分配唯一的**客户端 ID**。在后面的部分中将会用到此值，因此，请从“配置”选项卡复制此值。
- Adal.js 使用 OAuth 隐式流来与 Azure AD 通信。你必须通过执行以下操作，为应用程序启用隐式流：
    - 单击“管理清单”下载应用程序清单。
    - 打开清单，然后找到 `oauth2AllowImplicitFlow` 属性。将其值设置为 `true`。
    - 再次单击“管理清单”，以保存并上载应用程序清单。

## 2.安装 ADAL 并配置 SPA
将应用程序注册到 Azure AD 后，可以安装 adal.js 并编写标识相关的代码。

-	首先，使用 Package Manager Console 将 adal.js 添加到 TodoSPA 项目：
  - 下载 [adal.js](https://raw.githubusercontent.com/AzureAD/azure-activedirectory-library-for-js/master/lib/adal.js) 并将其添加到 `App/Scripts/` 项目目录。
  - 下载 [adal-angular.js](https://raw.githubusercontent.com/AzureAD/azure-activedirectory-library-for-js/master/lib/adal-angular.js) 并将其添加到 `App/Scripts/` 项目目录。
  - 在 `index.html` 将每个脚本加载到 `</body>` 的末尾之前：

js

	...
	<script src="App/Scripts/adal.js"></script>
	<script src="App/Scripts/adal-angular.js"></script>
	...


-	要使 SPA 后端待办事项列表 API 接受来自浏览器的令牌，后端需要有关应用程序注册的配置信息。在 TodoSPA 项目中，打开 `web.config`。替换 `<appSettings>` 中的元素值，以反映你在 Azure 门户中输入的值。只要使用 ADAL，你的代码就会引用这些值。
    -	`ida:Tenant` 是 Azure AD 租户的域，例如 contoso.partner.onmschina.cn
    -	`ida:Audience` 必须是从门户复制的应用程序的**客户端 ID**。

## 3.使用 ADAL 来保护 SPA 中的页面
Adal.js 能够与 AngularJS 路由和 http 提供程序集成，使你就可以在 SPA 中保护单个视图。

- 在 `App/Scripts/app.js` 中引入 adal.js 模块：

js

	angular.module('todoApp', ['ngRoute','AdalAngular'])
	.config(['$routeProvider','$httpProvider', 'adalAuthenticationServiceProvider',
	function ($routeProvider, $httpProvider, adalProvider) {
	...

- 现在，你可以使用应用程序注册的配置值初始化 `adalProvider`（同样在 `App/Scripts/app.js` 中）：

js

	adalProvider.init(
	{
		  instance: 'https://login.microsoftonline.com/',
		  tenant: 'Enter your tenant name here e.g. contoso.partner.onmschina.cn',
		  clientId: 'Enter your client ID here e.g. e9a5a8b6-8af7-4719-9821-0deef255f68e',
		  extraQueryParameter: 'nux=1',
		  //cacheLocation: 'localStorage', // enable this for IE, as sessionStorage does not work for localhost.
		},
		$httpProvider
	);

- 若要在应用程序中保护 `TodoList` 视图，只需编写一行代码 - `requireADLogin`。

js

	...
	}).when("/TodoList", {
		    controller: "todoListCtrl",
		    templateUrl: "/App/Views/TodoList.html",
		     requireADLogin: true,
	...


现在，你已获得一个安全的单页面应用程序，它可以让用户登录，并可向其后端 API 发出受持有者令牌保护的请求。当用户单击 `TodoList` 链接时，adal.js 会根据需要自动重定向到 Azure AD 以进行登录。此外，adal.js 会自动将 access\_token 附加到已发送至应用程序后端的任何 ajax 请求。以上是使用 adal.js 生成 SPA 的最低要求 - SPA 中还提供了其他许多有用的功能：

- 若要显式发出登录和注销请求，你可以在调用 adal.js 的控制器中定义函数。在 `App/Scripts/homeCtrl.js` 中：

js

		...
		$scope.login = function () {
		    adalService.login();
		};
		$scope.logout = function () {
		    adalService.logOut();
		};
		...

- 你可能还想要在应用程序的 UI 中显示用户信息。adal 服务已添加到 `userDataCtrl` 控制器，因此你可以在关联的视图 `App/Views/UserData.html` 中访问 `userInfo` 对象：

js

	<p>{{userInfo.userName}}</p>
	<p>aud:{{userInfo.profile.aud}}</p>
	<p>iss:{{userInfo.profile.iss}}</p>
	...


- 另外，在许多情况下，你想要知道用户是否已登录。你也可以使用 `userInfo` 对象来收集此信息。例如，在 `index.html` 中，可以根据身份验证状态显示“登录”或“注销”按钮：

js

	<li><a class="btn btn-link" ng-show="userInfo.isAuthenticated" ng-click="logout()">Logout</a></li>
	<li><a class="btn btn-link" ng-hide=" userInfo.isAuthenticated" ng-click="login()">Login</a></li>


祝贺你！ 你现已完成创建 Azure AD 集成的单页面应用程序。该应用程序可对用户进行身份验证，使用 OAuth 2.0 安全调用其后端，并获取有关用户的基本信息。如果你尚未这样做，可以在租户中填充一些用户。运行待办事项列表 SPA，然后使用这些用户之一进行登录。将任务添加到用户待办事项列表、注销和重新登录。

使用 Adal.js 可以方便地将所有这些常见标识功能合并到应用程序中。它会负责所有的繁琐工作 - 缓存管理、OAuth 协议支持、向用户显示登录名 UI、刷新已过期的令牌，等等。

[此处](https://github.com/AzureADQuickStarts/SinglePageApp-AngularJS-DotNet/archive/complete.zip)提供了已完成示例（无需配置值）供你参考。现在，你可以转到其他方案。你可能想要尝试：

[从 SPA 调用 CORS Web API >>](https://github.com/AzureAdSamples/SinglePageApp-WebAPI-AngularJS-DotNet)

[AZURE.INCLUDE [active-directory-devquickstarts-additional-resources](../../includes/active-directory-devquickstarts-additional-resources.md)]

<!---HONumber=Mooncake_0808_2016-->