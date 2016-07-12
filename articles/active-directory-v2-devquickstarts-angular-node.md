<properties
	pageTitle="Azure AD v2.0 AngularJS 入门 | Azure"
	description="如何构建一个使用个人 Microsoft 帐户和工作或学校帐户登录用户的 Angular JS 单页应用。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="02/20/2016"
	wacn.date="06/24/2016"/>


# 将登录凭据添加到 AngularJS 单页应用 - NodeJS

在本文中，我们将使用 Azure Active Directory v2.0 终结点将 Microsoft 支持的帐户的登录凭据添加到 AngularJS 应用。v2.0 终结点可让你在应用中执行单一集成，以及使用个人和工作/学校帐户对用户进行身份验证。

本示例是一个可在后端 REST API 存储任务的简单待办事项列表单页应用，它是使用 NodeJS 编写的，并使用 Azure AD 的 OAuth 持有者令牌进行保护。AngularJS 应用使用我们的开源 JavaScript 身份验证库 [adal.js](https://github.com/AzureAD/azure-activedirectory-library-for-js) 来处理整个登录过程，并获取用于调用 REST API 的令牌。可以应用与此相同的模式来验证其他 REST API，例如 [Microsoft Graph](https://graph.microsoft.com) 或 Azure Resource Manager API。

> [AZURE.NOTE]
	v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。

## 下载

若要开始，你需要下载并安装 [node.js](https://nodejs.org)。然后可以克隆或[下载](https://github.com/AzureADQuickStarts/AppModelv2-SinglePageApp-AngularJS-NodeJS/archive/skeleton.zip)骨架应用：


		git clone --branch skeleton https://github.com/AzureADQuickStarts/AppModelv2-SinglePageApp-AngularJS-NodeJS.git


该骨架应用包含简单 AngularJS 应用的重复使用代码，但是缺少与标识相关的所有部分。如果你不想要延用该应用，可以克隆或[下载](https://github.com/AzureADQuickStarts/AppModelv2-SinglePageApp-AngularJS-NodeJS/archive/complete.zip)完整的示例。


		git clone https://github.com/AzureADSamples/SinglePageApp-AngularJS-NodeJS.git


## 注册应用程序

首先，在[应用注册门户](https://apps.dev.microsoft.com)中创建应用，或者遵循以下[详细步骤](/documentation/articles/active-directory-v2-app-registration/)。请确保：

- 为应用程序添加 **Web** 平台。
- 输入正确的**重定向 URI**。本示例的默认值为 `http://localhost:8080`。
- 保留启用“允许隐式流”复选框。 

复制分配给应用程序的“应用程序 ID”，因为稍后将要用到。

## 安装 adal.js
若要开始，请导航到下载的项目并安装 adal.js。如果已安装 [bower](http://bower.io/)，只要运行以下命令即可。如有任何依赖版本不匹配的情况，只需选择较高的版本。

		bower install adal-angular#experimental


或者，你可以手动下载 [adal.js](https://raw.githubusercontent.com/AzureAD/azure-activedirectory-library-for-js/experimental/dist/adal.min.js) 和 [adal-angular.js](https://raw.githubusercontent.com/AzureAD/azure-activedirectory-library-for-js/experimental/dist/adal-angular.min.js)。将这两个文件添加到 `app/lib/adal-angular-experimental/dist` 目录。

现在，请在你偏爱的文本编辑器中打开该项目，并在页面正文的末尾加载 adal.js：

		html
		<!--index.html-->
		
		...
		
		<script src="App/bower_components/dist/adal.min.js"></script>
		<script src="App/bower_components/dist/adal-angular.min.js"></script>
		
		...


## 设置 REST API

在设置的同时，让我们查看后端 REST API 的工作方式。在命令提示符下，通过运行以下命令安装所有必要的包（确保你处于项目的顶层目录）：


		npm install


现在，请打开 `config.js` 并替换 `audience` 值：

		js
		exports.creds = {
		     
		     // TODO: Replace this value with the Application ID from the registration portal
		     audience: '<Your-application-id>',
			 
			 ...
		}


REST API 使用此值来验证发出 AJAX 请求时从 Angular 应用收到的令牌。请注意，这个简单的 REST API 会在内存中存储数据 - 因此，每次停止服务器后，你将会丢失以前创建的所有任务。

有关 REST API 工作方式的讨论到此为止。你可以自由摸索代码，但是如果想要深入了解如何使用 Azure AD 保护 Web API，请参阅[此文](/documentation/articles/active-directory-v2-devquickstarts-node-api/)。

## 登录用户
编写一些标识代码。你可能已发现 adal.js 包含 AngularJS 提供程序，该程序可以顺畅使用 Angular 路由机制。首先，将 adal 模块添加到应用：

		js
		// app/scripts/app.js
		
		angular.module('todoApp', ['ngRoute','AdalAngular'])
		.config(['$routeProvider','$httpProvider', 'adalAuthenticationServiceProvider',
		 function ($routeProvider, $httpProvider, adalProvider) {
		
		...


现在可以使用应用程序 ID 初始化 `adalProvider`：

		js
		// app/scripts/app.js
		
		...
		
		adalProvider.init({
		        
		        // Use this value for the public instance of Azure AD
		        instance: 'https://login.microsoftonline.com/', 
		        
		        // The 'common' endpoint is used for multi-tenant applications like this one
		        tenant: 'common',
		        
		        // Your application id from the registration portal
		        clientId: '<Your-application-id>',
		        
		        // If you're using IE, uncommment this line - the default HTML5 sessionStorage does not work for localhost.
		        //cacheLocation: 'localStorage',
		         
		    }, $httpProvider);


很好，现在 adal.js 有了保护应用和登录用户所需的所有信息。若要对应用中的特定路由强制登录，只需编写一行代码：

		js
		// app/scripts/app.js
		
		...
		
		}).when("/TodoList", {
		    controller: "todoListCtrl",
		    templateUrl: "/static/views/TodoList.html",
		    requireADLogin: true, // Ensures that the user must be logged in to access the route
		})
		
		...


现在，当用户单击 `TodoList` 链接时，adal.js 会根据需要自动重定向到 Azure AD 以进行登录。你也可以通过在控制器中调用 adal.js，显式发送登录和注销请求：

		js
		// app/scripts/homeCtrl.js
		
		angular.module('todoApp')
		// Load adal.js the same way for use in controllers and views   
		.controller('homeCtrl', ['$scope', 'adalAuthenticationService','$location', function ($scope, adalService, $location) {
		    $scope.login = function () {
		        
		        // Redirect the user to sign in
		        adalService.login();
		        
		    };
		    $scope.logout = function () {
		        
		        // Redirect the user to log out    
		        adalService.logOut();
		    
		    };
		...


## 显示用户信息
用户现已登录，你可能需要访问应用程序中已登录用户的身份验证数据。Adal.js 在 `userInfo` 对象中为你公开此信息。若要在视图中访问此对象，首先请将 adal.js 添加到相应控制器的根范围：

		js
		// app/scripts/userDataCtrl.js
		
		angular.module('todoApp')
		// Load ADAL for use in view
		.controller('userDataCtrl', ['$scope', 'adalAuthenticationService', function ($scope, adalService) {}]);


然后可以直接在视图中寻址 `userInfo` 对象：

		html
		<!--app/views/UserData.html-->
		
		...
		
		    <!--Get the user's profile information from the ADAL userInfo object-->
		    <tr ng-repeat="(key, value) in userInfo.profile">
		        <td>{{key}}</td>
		        <td>{{value}}</td>
		    </tr>
		...


也可以使用 `userInfo` 对象来确定用户是否已登录。
		
		html
		<!--index.html-->
		
		...
		
		    <!--Use the ADAL userInfo object to show the right login/logout button-->
		    <ul class="nav navbar-nav navbar-right">
		        <li><a class="btn btn-link" ng-show="userInfo.isAuthenticated" ng-click="logout()">Logout</a></li>
		        <li><a class="btn btn-link" ng-hide="userInfo.isAuthenticated" ng-click="login()">Login</a></li>
		    </ul>
		...


## 调用 REST API
最后，获取一些令牌并调用 REST API，以创建、读取、更新和删除任务。知道吗？ 你*什么事*都不用做。Adal.js 将自动为你获取、缓存和刷新令牌。它还会将这些令牌附加到发往 REST API 的传出 AJAX 请求。

到底是如何做到这一点的呢？ 一切归功于神奇的 [AngularJS 拦截器](https://docs.angularjs.org/api/ng/service/$http)，它可以让 adal.js 转换传出和传入的 http 消息。此外，adal.js 假设作为窗口发送到同一个域的任何请求都应该使用与 AngularJS 应用相同的应用程序 ID 所用的令牌。这就是为什么我们在 Angular 应用和 NodeJS REST API 中使用相同的应用程序 ID。当然，你可以重写此行为，并根据需要告知 adal.js 获取其他 REST API 的令牌 - 但是对于此简单方案，使用默认值即可。

下面代码段演示了如何轻松地从 Azure AD 发送包含持有者令牌的请求：

		js
		// app/scripts/todoListSvc.js
		
		...
		return $http.get('/api/tasks');
		...


祝贺你！ 你现已完成创建 Azure AD 集成的单页面应用。佩服吧！该应用可对用户进行身份验证，使用 OpenID Connect 安全调用其后端 REST API，并获取有关用户的基本信息。它原本就支持来自 Azure AD 的具有个人 Microsoft 帐户或工作/学校帐户的任何用户。运行以下命令以尝试使用该应用：


		node server.js


在浏览器中，导航到 `http://localhost:8080`。使用个人 Microsoft 帐户或工作/学校帐户登录。将任务添加到用户的待办事项列表，然后注销。尝试使用其他类型的帐户登录。如果你需要一个 Azure AD 租户来创建工作/学校用户，请[在此处了解如何获取租户](/documentation/articles/active-directory-howto-tenant/)（免费）。

如果要继续了解 v2.0 终结点，请返回到 [v2.0 开发人员指南](/documentation/articles/active-directory-appmodel-v2-overview/)。有关更多资源，请查看：

- [GitHub 上的 Azure 示例 >>](https://github.com/Azure-Samples)
- [堆栈溢出网站上的 Azure AD >>](http://stackoverflow.com/questions/tagged/azure-active-directory)
- [Azure.com](/documentation/services/identity/) 上的 Azure AD 文档 >>
<!---HONumber=Mooncake_0516_2016-->