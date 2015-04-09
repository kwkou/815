<properties pageTitle="如何启用 Apple 推送通知" metaKeywords="" description="按照本教程进行操作，创建使用 Azure 移动服务的新服务。" metaCanonical="" services="mobile-services,notification-hubs" documentationCenter="Mobile" title="How to enable Apple push notifications" authors="glenga" solutions="" manager="dwrede" editor="" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-ios" ms.devlang="multiple" ms.topic="article" ms.date="11/21/2014" ms.author="glenga" />

# 如何启用 Apple 推送通知

本主题演示如何使用 Apple 推送通知服务 (APNS) 为 iOS 应用程序启用推送通知。获取的证书用于在 [Azure 管理门户][Management Portal]中为 iOS 应用程序注册推送通知。有关包括对应用程序的更新的完整端到端教程，请参阅[推送通知入门]。 

[WACOM.INCLUDE [Enable Apple Push Notifications](../includes/enable-apple-push-notifications.md)]

现在，你可以使用此 .p12 证书允许服务使用 APNS 进行身份验证并代表你的应用程序发送推送通知。

<!-- Anchors. -->


<!-- Images. -->


<!-- URLs. -->
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-ios-get-started-push/
[移动服务 SDK]: https://go.microsoft.com/fwLink/p/?LinkID=268375

[Management Portal]: https://manage.windowsazure.cn/

