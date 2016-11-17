<properties
	pageTitle="使用 Azure 移动应用在 iOS 中添加身份验证"
	description="了解如何使用 Azure 移动应用通过各种标识提供者（包括 AAD 和 Microsoft）对 iOS 应用的用户进行身份验证。"
	services="app-service\mobile"
	documentationCenter="ios"
	authors="krisragh"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-ios"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="06/28/2016"
	wacn.date="09/26/2016"
	ms.author="yuaxu"/>

# Add authentication to your iOS app（将身份验证添加到 iOS 应用）

[AZURE.INCLUDE [app-service-mobile-selector-get-started-users](../../includes/app-service-mobile-selector-get-started-users.md)]

本教程介绍如何使用支持的标识提供者向 [iOS 快速入门]项目添加身份验证。本教程基于 [iOS quick start]（iOS 快速入门）教程，必须先完成该教程。

##<a name="register"></a>注册应用以进行身份验证并配置应用服务

[AZURE.INCLUDE [app-service-mobile-register-authentication](../../includes/app-service-mobile-register-authentication.md)]

##<a name="permissions"></a>将权限限制给已经过身份验证的用户

[AZURE.INCLUDE [app-service-mobile-restrict-permissions-dotnet-backend](../../includes/app-service-mobile-restrict-permissions-dotnet-backend.md)]

在 Xcode 中，按“运行”启动应用。将引发异常，因为应用尝试以未经身份验证的用户身份访问后端，但 _TodoItem_ 表现在要求身份验证。

##<a name="add-authentication"></a>向应用程序添加身份验证

[AZURE.INCLUDE [app-service-mobile-ios-authenticate-app](../../includes/app-service-mobile-ios-authenticate-app.md)]


<!-- URLs. -->

[iOS quick start]: /documentation/articles/app-service-mobile-ios-get-started/
[iOS 快速入门]: /documentation/articles/app-service-mobile-ios-get-started/

[Azure portal]: https://portal.azure.cn

<!---HONumber=Mooncake_0919_2016-->