<properties
	pageTitle="向应用程序添加推送通知 (iOS) | JavaScript 后端"
	description="了解如何使用 Azure 移动服务将推送通知发送到 iOS 应用程序。"
	services="mobile-services,notification-hubs"
	documentationCenter="ios"
	manager="dwrede"
	editor=""
	authors="krisragh"/>

<tags
	ms.service="mobile-services"
	ms.date="03/09/2016"
	wacn.date="04/11/2016"/>

#  向 iOS 应用程序和 JavaScript 后端添加推送通知

[AZURE.INCLUDE [mobile-services-selector-get-started-push](../includes/mobile-services-selector-get-started-push.md)]

本教程说明将推送通知发送到[快速入门项目](/documentation/articles/mobile-services-ios-get-started/)，这样，每次插入一条记录时，你的移动服务就会发送一条推送通知。你必须先完成[移动服务入门]教程。

> [AZURE.NOTE][IOS 模拟器不支持推送通知](https://developer.apple.com/zh-cn/library/ios/documentation/IDEs/Conceptual/iOS_Simulator_Guide/TestingontheiOSSimulator.html)，因此你必须使用物理 iOS 设备。你还需要付费注册 [Apple 开发人员计划成员身份](https://developer.apple.com/programs/ios/)。

[AZURE.INCLUDE [启用 Apple 推送通知](../includes/enable-apple-push-notifications.md)]


##  <a id="configure"></a>配置 Azure 以发送推送通知

[AZURE.INCLUDE [在 Azure 移动服务中配置推送通知](../includes/mobile-services-apns-configure-push.md)]

##  <a id="update-scripts"></a>更新后端脚本以发送推送通知

* 在 [Azure 管理门户]中，单击“数据”选项卡，然后单击“TodoItem”。在 **TodoItem** 中，单击“脚本”选项卡，然后选择“插入”。将显示当 **TodoItem** 表中发生插入时所调用的函数。

* 将 insert 函数替换为以下代码，然后单击“保存”。这将会注册一个新的插入脚本，该脚本使用 [apns 对象]将推送通知（插入的文本）发送到插入请求中提供的设备。此脚本将延迟发送通知，使你有足够的时间关闭应用程序以接收推送通知。


	```
        function insert(item, user, request) {
            request.execute();
            // Set timeout to delay the notification, to provide time for the
            // app to be closed on the device to demonstrate push notifications
            setTimeout(function() {
                push.apns.send(null, {
                    alert: "Alert: " + item.text,
                    payload: {
                        inAppMessage: "Hey, a new item arrived: '" + item.text + "'"
                    }
                });
            }, 2500);
        }
	```

[AZURE.INCLUDE [向应用程序添加推送通知](../includes/add-push-notifications-to-app.md)]

[AZURE.INCLUDE [在应用程序中测试推送通知](../includes/test-push-notifications-in-app.md)]


<!-- Anchors. -->


<!-- Images. -->
[5]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step5.png
[6]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step6.png
[7]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step7.png

[9]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step9.png
[10]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step10.png
[17]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step17.png
[18]: ./media/mobile-services-ios-get-started-push/mobile-services-selection.png
[19]: ./media/mobile-services-ios-get-started-push/mobile-push-tab-ios.png
[20]: ./media/mobile-services-ios-get-started-push/mobile-push-tab-ios-upload.png
[21]: ./media/mobile-services-ios-get-started-push/mobile-portal-data-tables.png
[22]: ./media/mobile-services-ios-get-started-push/mobile-insert-script-push2.png
[23]: ./media/mobile-services-ios-get-started-push/mobile-quickstart-push1-ios.png
[24]: ./media/mobile-services-ios-get-started-push/mobile-quickstart-push2-ios.png
[25]: ./media/mobile-services-ios-get-started-push/mobile-quickstart-push3-ios.png
[26]: ./media/mobile-services-ios-get-started-push/mobile-quickstart-push4-ios.png
[28]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-step18.png

[101]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-01.png
[102]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-02.png
[103]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-03.png
[104]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-04.png
[105]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-05.png
[106]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-06.png
[107]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-07.png
[108]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-08.png

[110]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-10.png
[111]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-11.png
[112]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-12.png
[113]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-13.png
[114]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-14.png
[115]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-15.png
[116]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-16.png
[117]: ./media/mobile-services-ios-get-started-push/mobile-services-ios-push-17.png

<!-- URLs.   -->
[Install Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[iOS Provisioning Portal]: http://go.microsoft.com/fwlink/p/?LinkId=272456
[Mobile Services iOS SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[Apple Push Notification Service]: http://go.microsoft.com/fwlink/p/?LinkId=272584
[移动服务入门]: /documentation/articles/mobile-services-ios-get-started/
[Get started with authentication]: /documentation/articles/mobile-services-ios-get-started-users/
[Azure 管理门户]: https://manage.windowsazure.cn/
[apns 对象]: http://go.microsoft.com/fwlink/p/?LinkId=272333

[Mobile Services server script reference]: /zh-cn/documentation/articles/mobile-services-how-to-use-server-scripts/

[Send push notifications to authenticated users]: /documentation/articles/mobile-services-javascript-backend-ios-push-notifications-app-users/
[What are Notification Hubs?]: /documentation/articles/notification-hubs-overview/
[Send broadcast notifications to subscribers]: /documentation/articles/notification-hubs-ios-send-breaking-news/
[Send template-based notifications to subscribers]: /documentation/articles/notification-hubs-ios-send-localized-breaking-news/
[Mobile Services Objective-C how-to conceptual reference]: /documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library/

<!---HONumber=Mooncake_0215_2016-->