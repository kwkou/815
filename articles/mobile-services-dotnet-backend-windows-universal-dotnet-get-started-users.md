<properties 
	pageTitle="身份验证入门（Windows 应用商店）| 移动开发人员中心" 
	description="了解如何使用移动服务通过提供各种标识提供程序（包括 Google、Facebook、Twitter 和 Microsoft）对 Windows 应用商店应用程序的用户进行身份验证。" 
	services="mobile-services" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="07/01/2015" 
	wacn.date="10/03/2015"/>

# 向移动服务应用程序添加身份验证 

[AZURE.INCLUDE [mobile-services-selector-get-started-users](../includes/mobile-services-selector-get-started-users.md)]

## 概述

本主题说明如何通过通用 Windows 应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。

本教程基于移动服务快速入门。此外，还必须先完成[移动服务入门]或[将移动服务添加到现有应用程序](/documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-data)教程。

>[AZURE.NOTE]本教程演示了如何对 Windows 应用商店和 Windows Phone 应用商店 8.1 应用中的用户使用服务器导向的身份验证。对于 Windows Phone 8.0 或 Windows Phone Silverlight 8.1 应用程序，请参阅此版本的[移动服务中的身份验证入门](/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-users)。有关客户端导向的身份验证的信息，请参阅[通过 Google、 Microsoft 和 Facebook SDK 登录 Azure 移动服务](http://azure.microsoft.com/blog/2014/10/27/logging-in-with-google-microsoft-and-facebook-sdks-to-azure-mobile-services/)。

##<a name="register"></a>注册应用程序以进行身份验证并配置移动服务

[AZURE.INCLUDE [mobile-services-register-authentication](../includes/mobile-services-register-authentication.md)]

[AZURE.INCLUDE [mobile-services-dotnet-backend-aad-server-extension](../includes/mobile-services-dotnet-backend-aad-server-extension.md)]

##<a name="permissions"></a>将权限限制给已经过身份验证的用户

[AZURE.INCLUDE [mobile-services-restrict-permissions-dotnet-backend](../includes/mobile-services-restrict-permissions-dotnet-backend.md)]

<ol start="6">
<li><p>在 Visual Studio 中，右键单击 TodoList 应用程序的 Windows 应用商店项目，然后单击“设为启动项目”<strong></strong>。</p></li>
<li><p>在共享的项目中，打开 App.xaml.cs 项目文件，找到 <a href="http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mobileservices.mobileserviceclient.aspx">MobileServiceClient</a> 的定义，并确保它已配置为连接到 Azure 中运行的移动服务。</p>
<p>请注意，使用 Visual Studio 工具将应用程序连接到移动服务时，该工具生成两组 <strong>MobileServiceClient</strong> 定义，每个客户端平台一组。这是简化生成的代码的好时机，你可以通过将 <code>#if...#endif</code> 包装的 <strong>MobileServiceClient</strong> 定义统一为单个解包的定义供这两个版本的应用程序使用来简化生成的代码。当你从 Azure 管理门户下载快速入门应用程序时无需执行此操作。</p>
</li> 
<li><p>按 F5 键运行该 Windows 应用商店应用程序；验证启动该应用程序后，是否会引发状态代码为 401（“未授权”）的未处理异常。</p>
   
   	<p>发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 <em>TodoItem</em> 表现在要求身份验证。</p></li>
</ol>

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

##<a name="add-authentication"></a>向应用程序添加身份验证

[AZURE.INCLUDE [mobile-services-windows-universal-dotnet-authenticate-app](../includes/mobile-services-windows-universal-dotnet-authenticate-app.md)]

>[AZURE.NOTE]如果已将 Windows 应用商店应用程序包信息注册到移动服务，则应该为 *useSingleSignOn* 参数提供 **true** 值以调用 <a href="http://go.microsoft.com/fwlink/p/?LinkId=311594" target="_blank">LoginAsync</a> 方法。如果不这样做，你的用户将继续显示登录提示每次调用 login 方法。

##<a name="tokens"></a>在客户端上存储授权令牌

[AZURE.INCLUDE [mobile-services-windows-store-dotnet-authenticate-app-with-token](../includes/mobile-services-windows-store-dotnet-authenticate-app-with-token.md)]


## <a name="next-steps"></a>后续步骤

在下一教程[移动服务用户的服务端授权][Authorize users with scripts]中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。

##另请参阅

+ [增强的用户功能](http://azure.microsoft.com/blog/2014/10/02/custom-login-scopes-single-sign-on-new-asp-net-web-api-updates-to-the-azure-mobile-services-net-backend/)<br/>你可以通过在 .NET 后端调用 **ServiceUser.GetIdentitiesAsync()** 方法，来获取标识提供者在你的移动服务中保留的其他用户数据。 

+ [移动服务 .NET 操作方法概念性参考]<br/>了解有关如何将移动服务与 .NET 客户端配合使用的详细信息。


<!-- Anchors. -->

[Register your app for authentication and configure Mobile Services]: #register
[Restrict table permissions to authenticated users]: #permissions
[Add authentication to the app]: #add-authentication
[Store authentication tokens on the client]: #tokens
[Next Steps]: #next-steps


<!-- URLs. -->

[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[Single sign-on for Windows Store apps by using Live Connect]: /documentation/articles/mobile-services-windows-store-dotnet-single-sign-on
[移动服务入门]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started
[Get started with data]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data
[Get started with authentication]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-users
[Get started with push notifications]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-push
[Authorize users with scripts]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-authorize-users-in-scripts
[JavaScript and HTML]: /documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-users
[Azure Management Portal]: https://manage.windowsazure.cn/
[移动服务 .NET 操作方法概念性参考]: /documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library
[Register your Windows Store app package for Microsoft authentication]: /documentation/articles/mobile-services-how-to-register-store-app-package-microsoft-authentication
<!---HONumber=71-->