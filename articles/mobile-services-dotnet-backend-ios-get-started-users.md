<properties
	pageTitle="向现有 Azure 移动服务应用添加身份验证 (iOS) | .NET 后端 | Microsoft Azure"
	description="了解如何使用移动服务通过各种标识提供程序（包括 Microsoft 和 Azure Active Directory）对 iOS 应用程序的用户进行身份验证。"
	services="mobile-services"
	documentationCenter="ios"
	authors="krisragh"
	manager="erikre"
	editor=""/>

<tags
	ms.service="mobile-services"
	ms.date="03/09/2016"
	wacn.date="04/18/2016"/>

# 向现有 Azure 移动服务应用程序添加身份验证

[AZURE.INCLUDE [mobile-services-selector-get-started-users](../includes/mobile-services-selector-get-started-users.md)]

在本教程中，你将要使用支持的标识提供程序向快速入门项目添加身份验证。本教程是在你必须先完成的[移动服务快速入门教程]的基础之上制作的。

##<a name="register"></a>注册应用程序以进行身份验证并配置移动服务

[AZURE.INCLUDE [mobile-services-register-authentication](../includes/mobile-services-register-authentication.md)]

[AZURE.INCLUDE [mobile-services-dotnet-backend-aad-server-extension](../includes/mobile-services-dotnet-backend-aad-server-extension.md)]

##<a name="permissions"></a>将权限限制给已经过身份验证的用户

[AZURE.INCLUDE [mobile-services-restrict-permissions-dotnet-backend](../includes/mobile-services-restrict-permissions-dotnet-backend.md)]

在 Xcode 中打开项目。按“运行”按钮启动应用程序。验证在应用程序启动后是否引发状态代码为 401（“未授权”）的异常。发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 _TodoItem_ 表现在要求身份验证。

##<a name="add-authentication"></a>向应用程序添加身份验证

[AZURE.INCLUDE [mobile-services-ios-authenticate-app](../includes/mobile-services-ios-authenticate-app.md)]

##<a name="store-authentication"></a>在应用程序中存储身份验证令牌

[AZURE.INCLUDE [mobile-services-ios-authenticate-app-with-token](../includes/mobile-services-ios-authenticate-app-with-token.md)]

##<a name="next-steps"></a>后续步骤

在下一教程[移动服务用户的服务端授权]中，你将使用用户 ID 值来筛选返回的数据。

<!-- Anchors. -->
[Register your app for authentication and configure Mobile Services]: #register
[Restrict table permissions to authenticated users]: #permissions
[Add authentication to the app]: #add-authentication
[Next Steps]: #next-steps
[Storing authentication tokens in your app]: #store-authentication

<!-- URLs. -->
[移动服务用户的服务端授权]: /documentation/articles/mobile-services-dotnet-backend-service-side-authorization/
[移动服务快速入门教程]: /documentation/articles/mobile-services-dotnet-backend-ios-get-started/
[Get started with authentication]: /documentation/articles/mobile-services-dotnet-backend-ios-get-started-users/
[Get started with push notifications]: /documentation/articles/mobile-services-dotnet-backend-ios-get-started-push/
[Authorize users with scripts]: /documentation/articles/mobile-services-dotnet-backend-service-side-authorization/

[Mobile Services .NET How-to Conceptual Reference]: /zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library
[Register your Windows Store app package for Microsoft authentication]: /documentation/articles/mobile-services-how-to-register-store-app-package-microsoft-authentication/

<!---HONumber=Mooncake_0215_2016-->