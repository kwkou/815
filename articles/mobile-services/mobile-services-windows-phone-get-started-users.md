<properties 
	pageTitle="身份验证入门 (Windows Phone) | Azure" 
	description="了解如何使用移动服务通过提供各种标识提供程序（包括 Microsoft 和 Azure Active Directory 对 Windows Phone 应用程序的用户进行身份验证。" 
	services="mobile-services" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="01/25/2016"
	wacn.date="03/21/2016"/>

#  向移动服务应用程序添加身份验证

[AZURE.INCLUDE [mobile-services-selector-get-started-users-legacy](../includes/mobile-services-selector-get-started-users-legacy.md)]

##概述

本主题说明如何通过应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。


本教程基于移动服务快速入门。你还必须先完成教程[将移动服务添加到现有应用程序]。

>[AZURE.NOTE]本教程演示如何管理移动服务使用各种标识提供程序的身份验证流。此方法易于配置，并支持多个提供者。通过使用客户端管理身份验证，你的应用程序有权访问所维护的标识提供程序的其他用户数据。你可以通过移动服务中获取相同的用户数据，通过调用 **user.getIdentities()** 服务器脚本中的函数。有关详细信息，请参阅[此文章](http://go.microsoft.com/fwlink/p/?LinkId=506605)。

## <a name="register"></a>注册应用程序以进行身份验证并配置移动服务


[AZURE.INCLUDE [mobile-services-register-authentication](../includes/mobile-services-register-authentication.md)]


## <a name="permissions"></a>将权限限制给已经过身份验证的用户


[AZURE.INCLUDE [mobile-services-restrict-permissions-javascript-backend](../includes/mobile-services-restrict-permissions-javascript-backend.md)]


&nbsp;&nbsp;&nbsp;&nbsp;3. 在 Visual Studio 中，打开在完成教程[将移动服务添加到现有应用](/documentation/articles/mobile-services-windows-phone-get-started-data/)时创建的项目。

&nbsp;&nbsp;&nbsp;&nbsp;4.按 F5 键运行这个基于快速入门的应用程序；验证启动该应用程序后，是否会引发状态代码为 401（“未授权”）的未处理异常。发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 *TodoItem* 表现在要求身份验证。

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

## <a name="add-authentication"></a>向应用程序添加身份验证

[AZURE.INCLUDE [mobile-services-windows-phone-authenticate-app](../includes/mobile-services-windows-phone-authenticate-app.md)]

## <a name="tokens"></a>在客户端上存储授权令牌

[AZURE.INCLUDE [mobile-services-windows-phone-authenticate-app-with-token](../includes/mobile-services-windows-phone-authenticate-app-with-token.md)]

##  <a name="next-steps"></a>后续步骤

在下一教程[移动服务用户的服务端授权](/documentation/articles/mobile-services-javascript-backend-service-side-authorization/)中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。

<!-- Anchors. -->
[Register your app for authentication and configure Mobile Services]: #register
[Restrict table permissions to authenticated users]: #permissions
[Add authentication to the app]: #add-authentication
[Next Steps]: #next-steps

<!-- Images. -->

[1]: ./media/mobile-services-wp8-get-started-users/mobile-services-selection.png
[2]: ./media/mobile-services-wp8-get-started-users/mobile-service-uri.png
[3]: ./media/mobile-services-wp8-get-started-users/mobile-identity-tab.png
[4]: ./media/mobile-services-wp8-get-started-users/mobile-portal-data-tables.png
[5]: ./media/mobile-services-wp8-get-started-users/mobile-portal-change-table-perms.png

<!-- URLs. -->
[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[将移动服务添加到现有应用程序]: /documentation/articles/mobile-services-windows-phone-get-started-data/
[Authorize users with scripts]: /documentation/articles/mobile-services-windows-phone-authorize-users-in-scripts/
 

<!---HONumber=Mooncake_0118_2016-->