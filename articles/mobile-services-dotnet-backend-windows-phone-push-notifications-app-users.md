<properties 
	pageTitle="向经过身份验证的用户发送推送通知" 
	description="了解如何向特定用户发送推送通知" 
	services="mobile-services,notification-hubs" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="02/26/2015" 
	wacn.date="06/26/2015"/>

# 向经过身份验证的用户发送推送通知



本主题说明如何向任何已注册设备上已经过身份验证的用户发送推送通知。与前面的[推送通知][Get started with push notifications]教程不同，本教程将指导你更改移动服务，以要求先对用户进行身份验证，然后，才可以将客户端注册到通知中心以发送推送通知。此外，你还要修改注册，以根据分配的用户 ID 添加标记。最后，将要更新服务器代码，以便只将通知发送到已经过身份验证的用户而不是所有注册。

本教程将指导你完成以下过程：

1. [更新服务以要求对注册进行身份验证]
2. [更新应用程序以要求在注册之前登录]
3. [测试应用程序]
 
本教程支持 Windows Phone 8.0 和 Windows Phone 8.1 Silverlight 应用程序。对于 Windows Phone 8.1 应用商店应用程序，请参阅[本主题的 Windows 应用商店版本](mobile-services-dotnet-backend-windows-store-dotnet-push-notifications-app-users)。

##先决条件 

在开始本教程之前，必须已完成以下移动服务教程：

+ [身份验证入门]<br/>向 TodoList 示例应用程序添加登录要求。

+ [推送通知入门]<br/>配置 TodoList 示例应用程序，以使用通知中心发送推送通知。

完成这两篇教程后，你可以防止未经身份验证的用户从你的移动服务注册推送通知。

##<a name="register"></a>更新服务以要求对注册进行身份验证

[AZURE.INCLUDE [mobile-services-dotnet-backend-push-notifications-app-users](../includes/mobile-services-dotnet-backend-push-notifications-app-users.md)]

##<a name="update-app"></a>更新应用程序以要求在注册之前登录

[AZURE.INCLUDE [mobile-services-windows-phone-push-notifications-app-users](../includes/mobile-services-windows-phone-push-notifications-app-users.md)]


##<a name="test"></a>测试应用程序

[AZURE.INCLUDE [mobile-services-windows-test-push-users](../includes/mobile-services-windows-test-push-users.md)]



<!-- Anchors. -->

[更新服务以要求对注册进行身份验证]: #register
[更新应用程序以要求在注册之前登录]: #update-app
[测试应用程序]: #test
[Next Steps]: #next-steps


<!-- URLs. -->

[身份验证入门]: mobile-services-dotnet-backend-windows-phone-get-started-users
[Get started with push notifications]: mobile-services-dotnet-backend-windows-phone-get-started-push
[推送通知入门]: mobile-services-dotnet-backend-windows-phone-get-started-push

[Azure Management Portal]: https://manage.windowsazure.cn/
[Mobile Services .NET How-to Conceptual Reference]: /documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library

<!---HONumber=61-->