<properties pageTitle="向经过身份验证的用户发送推送通知" metaKeywords="push notifications, authentication, users, Notification Hubs, Mobile Services" description="Learn how to send push notifications to specific " metaCanonical="" services="mobile-services,notification-hubs" documentationCenter="Mobile" title="Get started with authentication in Mobile Services" authors="glenga" solutions="Mobile" manager="dwrede" editor="" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-windows-store" ms.devlang="dotnet" ms.topic="article" ms.date="09/29/2014" ms.author="glenga" />

# 向经过身份验证的用户发送推送通知

[WACOM.INCLUDE [mobile-services-selector-push-users](../includes/mobile-services-selector-push-users.md)]

本主题说明如何向任何已注册设备上已经过身份验证的用户发送推送通知。与前面的[推送通知][推送通知入门]教程不同，本教程将指导你更改移动服务，以要求先对用户进行身份验证，然后，才可以将客户端注册到通知中心以发送推送通知。此外，你还要修改注册，以根据分配的用户 ID 添加标记。最后，将要更新服务器代码，以便只将通知发送到已经过身份验证的用户而不是所有注册。

本教程将指导你完成以下过程：

+ [更新服务以要求对注册进行身份验证]
+ [更新应用程序以要求在注册之前登录]
+ [测试应用程序]
 
本教程支持 Windows 应用商店和 Windows Phone 应用商店应用程序。

##先决条件 

在开始本教程之前，必须已完成以下移动服务教程：

+ [身份验证入门]<br/>向 TodoList 示例应用程序添加登录要求。

+ [推送通知入门]<br/>配置 TodoList 示例应用程序，以使用通知中心发送推送通知。 

完成这两篇教程后，你可以防止未经身份验证的用户从你的移动服务注册推送通知。

##<a name="register"></a>更新服务以要求对注册进行身份验证

[WACOM.INCLUDE [mobile-services-dotnet-backend-push-notifications-app-users](../includes/mobile-services-dotnet-backend-push-notifications-app-users.md)] 

##<a name="update-app"></a>更新应用程序以要求在注册之前登录

[WACOM.INCLUDE [mobile-services-windows-store-dotnet-push-notifications-app-users](../includes/mobile-services-windows-store-dotnet-push-notifications-app-users.md)] 

##<a name="test"></a>测试应用程序

[WACOM.INCLUDE [mobile-services-windows-test-push-users](../includes/mobile-services-windows-test-push-users.md)] 

<!---## <a name="next-steps"> </a>后续步骤

在下一篇教程[移动服务用户的服务端授权][使用脚本向用户授权]中，你将使用移动服务基于已经过身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。请在[移动服务 .NET 操作方法概念性参考]中了解有关如何将移动服务与 .NET 一起使用的详细信息-->

<!-- Anchors. -->
[更新服务以要求对注册进行身份验证]： #register
[更新应用程序以要求在注册之前登录]： #update-app
[测试应用程序]： #test
[后续步骤]：#next-steps


<!-- URLs. -->
[身份验证入门]：/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-users/
[推送通知入门]：/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-push/

[Azure 管理门户]：https://manage.windowsazure.cn/
[移动服务 .NET 操作方法概念性参考]：/zh-cn/develop/mobile/how-to-guides/work-with-net-client-library
