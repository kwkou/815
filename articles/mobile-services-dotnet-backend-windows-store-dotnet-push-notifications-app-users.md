<properties 
	pageTitle="向经过身份验证的用户发送推送通知（通用 Windows 8.1）| Azure 移动服务" 
	description="了解如何使用 Azure 移动服务向运行通用 Windows 8.1 应用并经过身份验证的特定用户发送推送通知。" 
	services="mobile-services,notification-hubs" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="03/05/2016"
	wacn.date="06/13/2016"/>

# 向经过身份验证的用户发送推送通知
[AZURE.INCLUDE [mobile-services-selector-push-users](../includes/mobile-services-selector-push-users.md)]


##概述

本主题说明如何向任何已注册设备上已经过身份验证的用户发送推送通知。与前面的[推送通知][Get started with push notifications]教程不同，本教程将指导你更改移动服务，以要求先对用户进行身份验证，然后，才可以将客户端注册到通知中心以发送推送通知。此外，你还要修改注册，以根据分配的用户 ID 添加标记。最后，将要更新服务器代码，以便只将通知发送到已经过身份验证的用户而不是所有注册。
 
本教程支持 Windows 应用商店和 Windows Phone 应用商店应用程序。

##先决条件 

在开始本教程之前，必须已完成以下移动服务教程：

+ [身份验证入门]  
向 TodoList 示例应用添加登录要求。

+ [推送通知入门]  
配置 TodoList 示例应用，以使用通知中心发送推送通知。

完成这两篇教程后，你可以防止未经身份验证的用户从你的移动服务注册推送通知。

##<a name="register"></a>更新服务以要求对注册进行身份验证

[AZURE.INCLUDE [mobile-services-dotnet-backend-push-notifications-app-users](../includes/mobile-services-dotnet-backend-push-notifications-app-users.md)]

##<a name="update-app"></a>更新应用程序以要求在注册之前登录

[AZURE.INCLUDE [mobile-services-windows-store-dotnet-push-notifications-app-users](../includes/mobile-services-windows-store-dotnet-push-notifications-app-users.md)]

##<a name="test"></a>测试应用程序

[AZURE.INCLUDE [mobile-services-windows-test-push-users](../includes/mobile-services-windows-test-push-users.md)]


<!-- Anchors. -->
[Updating the service to require authentication for registration]: #register
[Updating the app to log in before registration]: #update-app
[Testing the app]: #test
[Next Steps]: #next-steps


<!-- URLs. -->
[身份验证入门]: /documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-users/
[Get started with push notifications]: /documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-push/
[推送通知入门]: /documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-push/

<!---HONumber=Mooncake_0215_2016-->