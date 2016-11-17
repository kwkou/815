<properties
	pageTitle="使用 Azure 移动应用将推送通知添加到 iOS 应用"
	description="了解如何使用 Azure 移动应用将推送通知发送到 iOS 应用。"
	services="app-service\mobile"
	documentationCenter="ios"
	manager="dwrede"
	editor=""
	authors="krisragh"/>

<tags
	ms.service="app-service-mobile"
	ms.date="06/28/2016"
	wacn.date="09/26/2016"/>


# 将推送通知添加到 iOS 应用

[AZURE.INCLUDE [app-service-mobile-selector-get-started-push](../../includes/app-service-mobile-selector-get-started-push.md)]

## 概述
本教程介绍如何向 [iOS 快速入门]项目添加推送通知，以便每次插入一条记录时，发送一条推送通知。本教程基于 [iOS quick start]（iOS 快速入门）教程，必须先完成该教程。如果不使用下载的快速入门服务器项目，必须将推送通知扩展包添加到项目。有关服务器扩展包的详细信息，请参阅 [Work with the .NET backend server SDK for Azure Mobile Apps](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/)（使用适用于 Azure 移动应用的 .NET 后端服务器 SDK）。

iOS 模拟器不支持推送通知，因此在本教程中，需要有一个 iOS 物理设备和 [Apple Developer Program 会员资格](https://developer.apple.com/programs/ios/)。

##<a name="create-hub"></a>创建通知中心

[AZURE.INCLUDE [app-service-mobile-create-notification-hub](../../includes/app-service-mobile-create-notification-hub.md)]

## <a id="register"></a>为推送通知注册应用

[AZURE.INCLUDE [启用 Apple 推送通知](../../includes/enable-apple-push-notifications.md)]

## 配置 Azure 以发送推送通知

[AZURE.INCLUDE [app-service-mobile-apns-configure-push](../../includes/app-service-mobile-apns-configure-push.md)]

##<a id="update-server"></a>更新服务器项目以发送推送通知

[AZURE.INCLUDE [app-service-mobile-dotnet-backend-configure-push-apns](../../includes/app-service-mobile-dotnet-backend-configure-push-apns.md)]

## <a id="add-push"></a>将推送通知添加到应用

[AZURE.INCLUDE [app-service-mobile-add-push-notifications-to-ios-app.md](../../includes/app-service-mobile-add-push-notifications-to-ios-app.md)]

## <a id="test"></a>在应用中测试推送通知

[AZURE.INCLUDE [在应用程序中测试推送通知](../../includes/test-push-notifications-in-app.md)]

##<a id="more"></a>其他信息

* 使用模板可以灵活地发送跨平台推送和本地化推送。[How to Use iOS Client Library for Azure Mobile Apps](/documentation/articles/app-service-mobile-ios-how-to-use-client-library/#templates)（如何使用适用于 Azure 移动应用的 iOS 客户端库）说明了如何注册模板。
* 使用标记可以锁定接收推送通知的细分客户。[Work with the .NET backend server SDK for Azure Mobile Apps](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/#tags)（使用适用于 Azure 移动应用的 .NET 后端服务器 SDK）说明了如何在设备安装中添加标记。

<!-- Anchors.  -->
[Generate iOS certificate signing request]: #certificates
[Register your app and enable push notifications]: #register
[Create a provisioning profile for the app]: #profile
[Add push notifications to the app]: #add-push
[Configure your mobile backend to send push requests]: #configure
[Update the server to send push notifications]: #update-server
[Publish the mobile backend to Azure]: #publish-mobile-service
[Test your app]: #test-the-service

<!-- Images. -->

<!-- URLs. -->
[iOS quick start]: /documentation/articles/app-service-mobile-ios-get-started/
[iOS 快速入门]: /documentation/articles/app-service-mobile-ios-get-started/

<!---HONumber=Mooncake_0919_2016-->