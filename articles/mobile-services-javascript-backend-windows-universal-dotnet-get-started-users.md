<properties 
	pageTitle="向通用 Windows 8.1 应用添加身份验证 | Azure 移动服务"
	description="了解如何使用移动服务通过提供各种标识提供程序（包括 Google、Facebook、Twitter 和 Microsoft）对 Windows 应用商店应用程序的用户进行身份验证。" 
	services="mobile-services" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="09/14/2015" 
	wacn.date="10/22/2015"/>

# 向通用 Windows 8.1 应用添加身份验证

[AZURE.INCLUDE [mobile-services-selector-get-started-users](../includes/mobile-services-selector-get-started-users.md)]

本主题说明如何通过通用 Windows 8.1 应用对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。

本教程基于移动服务快速入门。此外，还必须先完成[移动服务入门]或[将移动服务添加到现有应用程序](/documentation/articles/mobile-services-javascript-backend-windows-universal-dotnet-get-started-data)教程。

>[AZURE.NOTE]本教程演示了如何对 Windows 应用商店和 Windows Phone 应用商店 8.1 应用程序中的用户进行身份验证。对于 Windows Phone 8.0 或 Windows Phone Silverlight 8.1 应用程序，请参阅此版本的[移动服务中的身份验证入门](/documentation/articles/mobile-services-windows-phone-get-started-users)。

>本教程演示如何使用各种标识提供者执行服务托管的身份验证流。此方法易于配置，并支持多个提供者。若要在 Windows 应用商店应用程序中改用 Live SDK 执行客户端托管的流，请参阅主题[使用 Microsoft 帐户以客户端托管身份验证方式对 Windows 应用商店应用程序进行身份验证](/documentation/articles/mobile-services-windows-store-dotnet-single-sign-on)。

## <a name="register"></a>注册应用程序以进行身份验证并配置移动服务

[AZURE.INCLUDE [mobile-services-register-authentication](../includes/mobile-services-register-authentication.md)]

## <a name="permissions"></a>将权限限制给已经过身份验证的用户

[AZURE.INCLUDE [mobile-services-restrict-permissions-windows](../includes/mobile-services-restrict-permissions-windows.md)]

>[AZURE.NOTE]当你使用 Visual Studio 工具将应用程序连接到移动服务时，该工具将生成两组 **MobileServiceClient** 定义，每个客户端平台一组。这是简化生成的代码的好时机，你可以通过将 `#if...#endif` 包装的 **MobileServiceClient** 定义统一为单个解包的定义供这两个版本的应用程序使用来简化生成的代码。如果从 Azure 管理门户下载快速入门应用程序，则无需执行此操作。

## <a name="add-authentication"></a>向应用程序添加身份验证

[AZURE.INCLUDE [mobile-services-windows-universal-dotnet-authenticate-app](../includes/mobile-services-windows-universal-dotnet-authenticate-app.md)]

现在，已由受信任标识提供者进行身份验证的任何用户都可访问 *TodoItem* 表。为了进一步提高用户特定数据的安全性，你还必须实施授权。为此，需要获取给定用户的用户 ID，然后根据该 ID 来确定该用户对给定的资源具有哪种级别的访问权限。

## <a name="tokens"></a>在客户端上存储授权令牌

[AZURE.INCLUDE [mobile-services-windows-store-dotnet-authenticate-app-with-token](../includes/mobile-services-windows-store-dotnet-authenticate-app-with-token.md)]

##  <a name="next-steps"></a>后续步骤

在下一教程[移动服务用户的服务端授权](/documentation/articles/mobile-services-javascript-backend-service-side-authorization)中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。

## 另请参阅

+ [增强的用户功能](http://go.microsoft.com/fwlink/p/?LinkId=506605)<br/>
你可以通过调用服务器脚本中的 **user.getIdentities()** 函数，来获取标识提供者在你的移动服务中保留的其他用户数据。 

+ [移动服务 .NET 操作方法概念性参考]<br/>
了解有关如何将移动服务与 .NET 客户端配合使用的详细信息。


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

[移动服务入门]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started
[Get started with data]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-data
[Get started with authentication]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-users
[Get started with push notifications]: vmobile-services-javascript-backend-windows-store-dotnet-get-started-push
[Authorize users with scripts]: /documentation/articles/mobile-services-windows-store-dotnet-authorize-users-in-scripts
[JavaScript and HTML]: /documentation/articles/mobile-services-windows-store-javascript-get-started-users
[Azure Management Portal]: https://manage.windowsazure.cn/
[移动服务 .NET 操作方法概念性参考]: /documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library
[Register your Windows Store app package for Microsoft authentication]: /documentation/articles/mobile-services-how-to-register-store-app-package-microsoft-authentication

<!---HONumber=74-->