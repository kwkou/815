<properties 
	pageTitle="身份验证入门 (Windows Phone) | 移动开发人员中心" 
	description="了解如何使用移动服务通过提供各种标识提供程序（包括 Google、Facebook、Twitter 和 Microsoft）对 Windows Phone 应用程序的用户进行身份验证。" 
	services="mobile-services" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="07/21/2015" 
	wacn.date="10/03/2015"/>

# 向移动服务应用程序添加身份验证

[AZURE.INCLUDE [mobile-services-selector-get-started-users](../includes/mobile-services-selector-get-started-users.md)]

## 概述

本主题说明如何通过 Windows Phone 应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。

>[AZURE.NOTE]本主题仅支持 Windows Phone 8.0 和 Windows Phone 8.1 Silverlight 应用程序。如果你有 Windows Phone 应用商店 8.1 或通用 Windows 应用程序，请遵循[本主题的通用 Windows 应用程序版本](/documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-users)。

本教程基于移动服务快速入门。此外，还必须先完成[移动服务入门]教程。

## 注册应用程序以进行身份验证并配置移动服务

[AZURE.INCLUDE [mobile-services-register-authentication](../includes/mobile-services-register-authentication.md)]

[AZURE.INCLUDE [mobile-services-dotnet-backend-aad-server-extension](../includes/mobile-services-dotnet-backend-aad-server-extension.md)]

##  将权限限制给已经过身份验证的用户

[AZURE.INCLUDE [mobile-services-restrict-permissions-dotnet-backend](../includes/mobile-services-restrict-permissions-dotnet-backend.md)]

&nbsp;&nbsp;6.在 Visual Studio 中打开你的客户端应用程序项目，并确保在 App.xaml.cs 中，**MobileServiceClient** 的实例已配置为使用移动服务的云 URL。

&nbsp;&nbsp;7.按 F5 键运行这个基于快速入门的应用程序；验证启动该应用程序后，是否会引发状态代码为 401（“未授权”）的未处理异常。当应用程序尝试以未经身份验证的用户身份访问移动服务，但 *TodoItem* 表现在要求身份验证时，就会发生此异常。

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

## 向应用程序添加身份验证

[AZURE.INCLUDE [mobile-services-windows-phone-authenticate-app](../includes/mobile-services-windows-phone-authenticate-app.md)]

## 在客户端上存储授权令牌

[AZURE.INCLUDE [mobile-services-windows-phone-authenticate-app-with-token](../includes/mobile-services-windows-phone-authenticate-app-with-token.md)]

##  后续步骤

在下一教程[移动服务用户的服务端授权][Authorize users with scripts]中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。请在[移动服务 .NET 操作方法概念性参考]中了解有关如何将移动服务与 .NET 一起使用的详细信息

<!-- Anchors. -->


<!-- URLs. -->

[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[Single sign-on for Windows Phone apps by using Live Connect]: /documentation/articles/mobile-services-windows-phone-single-sign-on
[移动服务入门]: /documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started
[Get started with data]: /documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-data
[Get started with authentication]: /documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-users
[Get started with push notifications]: /documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-push
[Authorize users with scripts]: /documentation/articles/mobile-services-dotnet-backend-windows-phone-authorize-users-in-scripts
[JavaScript and HTML]: /documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-users
[Azure Management Portal]: https://manage.windowsazure.cn/
[移动服务 .NET 操作方法概念性参考]: /documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library
[Register your Windows Store app package for Microsoft authentication]: /documentation/articles/mobile-services-how-to-register-store-app-package-microsoft-authentication
<!---HONumber=71-->