<properties
	pageTitle="使用 Azure 移动应用将推送通知添加到 iOS 应用"
	description="了解如何使用 Azure 移动应用将推送通知发送到 iOS 应用。"
	services="app-service\mobile"
	documentationCenter="ios"
	manager="yochayk"
	editor=""
	authors="yuaxu"/>  


<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-ios"
	ms.devlang="objective-c"
	ms.topic="article"
	ms.date="10/10/2016"
	wacn.date="12/26/2016"
	ms.author="yuaxu"/>


# 将推送通知添加到 iOS 应用

[AZURE.INCLUDE [app-service-mobile-selector-get-started-push](../../includes/app-service-mobile-selector-get-started-push.md)]

## 概述
本教程介绍如何向 [iOS 快速入门]项目添加推送通知，以便每次插入一条记录时，向设备发送一条推送通知。

如果不使用下载的快速入门服务器项目，则需要推送通知扩展包。有关详细信息，请参阅[使用适用于 Azure 移动应用的 .NET 后端服务器 SDK](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk)。

[iOS 模拟器不支持推送通知](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/iOS_Simulator_Guide/TestingontheiOSSimulator.html)。用户需要 iOS 物理设备和 [Apple 开发人员计划成员身份](https://developer.apple.com/programs/ios/)。

## <a name="configure-hub"></a>配置通知中心
[AZURE.INCLUDE [app-service-mobile-configure-notification-hub](../../includes/app-service-mobile-configure-notification-hub.md)]

## <a id="register"></a>为推送通知注册应用

[AZURE.INCLUDE [启用 Apple 推送通知](../../includes/enable-apple-push-notifications.md)]

## 配置 Azure 以发送推送通知

[AZURE.INCLUDE [app-service-mobile-apns-configure-push](../../includes/app-service-mobile-apns-configure-push.md)]

## <a id="update-server"></a>更新后端以发送推送通知

[AZURE.INCLUDE [app-service-mobile-dotnet-backend-configure-push-apns](../../includes/app-service-mobile-dotnet-backend-configure-push-apns.md)]

## <a id="add-push"></a>将推送通知添加到应用

[AZURE.INCLUDE [app-service-mobile-add-push-notifications-to-ios-app.md](../../includes/app-service-mobile-add-push-notifications-to-ios-app.md)]

## <a id="test"></a>测试推送通知

[AZURE.INCLUDE [在应用程序中测试推送通知](../../includes/test-push-notifications-in-app.md)]

##<a id="more"></a>其他信息

* 使用模板可以灵活地发送跨平台推送和本地化推送。[How to Use iOS Client Library for Azure Mobile Apps](/documentation/articles/app-service-mobile-ios-how-to-use-client-library/#templates)（如何使用适用于 Azure 移动应用的 iOS 客户端库）说明了如何注册模板。

<!-- Anchors.  -->


<!-- Images. -->

<!-- URLs. -->
[iOS 快速入门]: /documentation/articles/app-service-mobile-ios-get-started/

<!---HONumber=Mooncake_1219_2016-->