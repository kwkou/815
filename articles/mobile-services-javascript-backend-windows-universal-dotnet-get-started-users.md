<properties 
	pageTitle="向通用 Windows 8.1 应用添加身份验证 | Azure 移动服务"
	description="了解如何使用移动服务通过提供各种标识提供程序（包括 Microsoft 和 Azure Active Directory）对 Windows 应用商店应用程序的用户进行身份验证。" 
	services="mobile-services" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="03/05/2016" 
	wacn.date="04/11/2016"/>

# 向通用 Windows 8.1 应用添加身份验证

[AZURE.INCLUDE [mobile-services-selector-get-started-users](../includes/mobile-services-selector-get-started-users.md)]


本主题说明如何通过通用 Windows 8.1 应用对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。

本教程基于移动服务快速入门。此外，还必须先完成[移动服务入门]教程。

>[AZURE.NOTE]本教程演示了如何对 Windows 应用商店和 Windows Phone 应用商店 8.1 应用程序中的用户进行身份验证。对于 Windows Phone 8.0 或 Windows Phone Silverlight 8.1 应用程序，请参阅此版本的[移动服务中的身份验证入门](/documentation/articles/mobile-services-windows-phone-get-started-users/)。

## <a name="register"></a>注册应用程序以进行身份验证并配置移动服务

[AZURE.INCLUDE [mobile-services-register-authentication](../includes/mobile-services-register-authentication.md)]

## <a name="permissions"></a>将权限限制给已经过身份验证的用户

[AZURE.INCLUDE [mobile-services-restrict-permissions-windows](../includes/mobile-services-restrict-permissions-windows.md)]

>[AZURE.NOTE] 当你使用 Visual Studio 工具将应用程序连接到移动服务时，该工具将生成两组 **MobileServiceClient** 定义，每个客户端平台一组。这是简化生成的代码的好时机，你可以通过将 `#if...#endif` 包装的 **MobileServiceClient** 定义统一为单个解包的定义供这两个版本的应用程序使用来简化生成的代码。如果你从 [Azure 经典门户]下载了快速入门应用，则无需执行此操作。

## <a name="add-authentication"></a>向应用程序添加身份验证

[AZURE.INCLUDE [mobile-services-windows-universal-dotnet-authenticate-app](../includes/mobile-services-windows-universal-dotnet-authenticate-app.md)]

现在，已由受信任标识提供者进行身份验证的任何用户都可访问 *TodoItem* 表。为了进一步提高用户特定数据的安全性，你还必须实施授权。为此，需要获取给定用户的用户 ID，然后根据该 ID 来确定该用户对给定的资源具有哪种级别的访问权限。

## <a name="tokens"></a>在客户端上存储授权令牌

先前示例显示标准登录，这要求在此应用每次启动时客户端同时联系标识提供商和移动服务。此方法不仅效率低下，而且如果很多客户尝试同时启动你的应用程序，你会遇到关于使用率的问题。更好的方法是缓存移动服务返回的授权令牌，然后在使用基于提供商的登录之前首先尝试使用此令牌。

>[AZURE.NOTE]无论你使用客户端管理的还是服务管理的身份验证，都可以缓存由移动服务颁发的令牌。本教程使用服务管理的身份验证。

[AZURE.INCLUDE [mobile-windows-universal-dotnet-authenticate-app-with-token](../includes/mobile-windows-universal-dotnet-authenticate-app-with-token.md)]

##  <a name="next-steps"></a>后续步骤

在下一教程[移动服务用户的服务端授权](/documentation/articles/mobile-services-javascript-backend-service-side-authorization/)中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。

## 另请参阅

+ [增强的用户功能](http://go.microsoft.com/fwlink/p/?LinkId=506605)<br/>
你可以通过调用服务器脚本中的 **user.getIdentities()** 函数，来获取标识提供者在你的移动服务中保留的其他用户数据。 


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

[移动服务入门]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started/
[Get started with authentication]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-users/
[Get started with push notifications]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push/
[Authorize users with scripts]: /documentation/articles/mobile-services-windows-store-dotnet-authorize-users-in-scripts/

[Azure 经典门户]: https://manage.windowsazure.cn/
[移动服务 .NET 操作方法概念性参考]: /documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library/
[Register your Windows Store app package for Microsoft authentication]: /documentation/articles/mobile-services-how-to-register-store-app-package-microsoft-authentication/

<!---HONumber=Mooncake_0118_2016-->