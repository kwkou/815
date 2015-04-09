<properties urlDisplayName="Get Started with Push (iOS)" pageTitle="开始使用推送通知 (iOS) |移动开发人员中心" metaKeywords="" description="了解如何使用 Azure 移动服务向 iOS 应用程序发送推送通知。" metaCanonical="http://www.windowsazure.cn/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push/" services="mobile-services,notification-hubs" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services" solutions="" manager="dwrede" editor="" authors="krisragh" />


# 向移动服务应用程序添加推送通知

[WACOM.INCLUDE [mobile-services-selector-get-started-push](../includes/mobile-services-selector-get-started-push.md)]

本主题演示如何使用 Azure 移动服务并通过 Apple 推送通知服务 (APNS) 向 iOS 应用程序发送推送通知。在本教程中，你将要使用 Azure 通知中心为[快速入门项目](/zh-cn/documentation/articles/mobile-services-ios-get-started/)启用推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。

本教程将指导你完成启用推送通知的以下基本步骤：

1. [生成证书签名请求](#certificates)
2. [注册应用程序和启用推送通知](#register)
3. [为应用程序创建配置文件](#profile)
4. [配置移动服务](#configure)
5. [向应用程序添加推送通知](#add-push)
6. [更新脚本以发送推送通知](#update-scripts)
7. [插入数据以接收通知](#test)

本教程需要的内容如下：

+ [移动服务 iOS SDK]
+ [XCode 4.5][安装 Xcode]
+ 支持 iOS 6.0（或更高版本）的设备
+ iOS 开发人员计划成员身份

   > [AZURE.NOTE] 由于推送通知配置要求，你必须在支持 iOS 的设备（iPhone 或 iPad），而不是在模拟器上部署和测试推送通知。

本教程基于移动服务快速入门。在开始本教程之前，您首先必须完成[移动服务入门]或[向现有应用程序添加移动服务][数据处理入门]。


[WACOM.INCLUDE [Enable Apple Push Notifications](../includes/enable-apple-push-notifications.md)]


## <a id="configure"></a>配置移动服务以发送推送请求

[WACOM.INCLUDE [mobile-services-apns-configure-push](../includes/mobile-services-apns-configure-push.md)]

## <a id="update-scripts"></a>在管理门户中更新已注册的插入脚本

1. 在管理门户中，单击**数据**选项卡，然后单击**TodoItem**表。

   ![][21]

2. 在 **todoitem** 中，单击**脚本**选项卡，然后选择**插入**。

  	![][22]

   将显示当 **TodoItem** 表中发生插入时所调用的函数。

3. 将 insert 函数替换为以下代码，然后单击"保存"：

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

   	这将会注册一个新的插入脚本，该脚本使用[apns 对象]将推送通知（插入的文本）发送到插入请求中提供的设备。


   > [AZURE.NOTE] 此脚本将延迟发送通知，使你有足够的时间关闭应用程序以接收推送通知。

## <a id="add-push"></a>向应用程序添加推送通知

1. 在 QSAppDelegate.m 中，插入以下代码段以便导入移动服务 iOS SDK：

        #import <WindowsAzureMobileServices/WindowsAzureMobileServices.h>

2. 在 QSAppDelegate.m 中，替换实施中的以下处理程序方法：

        - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:
        (NSDictionary *)launchOptions
        {
            // Register for remote notifications
            [[UIApplication sharedApplication] registerForRemoteNotificationTypes:
            UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];
            return YES;
        }

3. 在 QSAppDelegate.m 中，在实施中添加以下处理程序方法：请确保复制移动服务 URL 和应用程序密钥值并切换它们中的占位符：

        - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:
        (NSData *)deviceToken {
            // TODO: update @"MobileServiceUrl" and @"AppKey" placeholders
			MSClient *client = [MSClient clientWithApplicationURLString:@"MobileServiceUrl" applicationKey:@"AppKey"];

            [client.push registerNativeWithDeviceToken:deviceToken tags:@[@"uniqueTag"] completion:^(NSError *error) {
                if (error != nil) {
                    NSLog(@"Error registering for notifications: %@", error);
                }
            }];
        }

4. 在 QSAppDelegate.m 中，在实施中添加以下处理程序方法：

        // Handle any failure to register.
        - (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:
        (NSError *)error {
            NSLog(@"Failed to register for remote notifications: %@", error);
        }

5. 在 QSAppDelegate.m 中，在实施中添加以下处理程序方法：  

        // Because alerts don't work when the app is running, the app handles them.
        // This uses the userInfo in the payload to display a UIAlertView.
        - (void)application:(UIApplication *)application didReceiveRemoteNotification:
        (NSDictionary *)userInfo {
            NSLog(@"%@", userInfo);
            UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Notification" message:
            [userInfo objectForKey:@"inAppMessage"] delegate:nil cancelButtonTitle:
            @"OK" otherButtonTitles:nil, nil];
            [alert show];
        }

您的应用现已更新，可支持推送通知。

## <a id="test"></a>在应用程序中测试推送通知

1. 在支持 iOS 的设备中按**运行**按钮以生成项目并启动应用程序，然后单击**确定**接受推送通知

  	![][23]

    > [AZURE.NOTE] 你必须显式接受来自应用程序的推送通知。此请求只会在首次运行应用程序时出现。

2. 在应用程序中键入有意义的文本（例如 A new Mobile Services task），然后单击加号(**+**)图标。

  	![][24]

3. 检查是否已收到通知，然后单击**确定**以取消通知。

  	![][25]

4. 重复步骤 2 并立即关闭应用程序，然后检查是否已显示以下推送通知。

  	![][26]

你已成功完成本教程。

## <a id="next-steps"></a>后续步骤

本教程演示了启用 iOS 应用程序以使用移动服务和通知中心发送推送通知的基础知识。接下来，请考虑完成以下教程之一：

+ [向经过身份验证的用户发送推送通知]
	<br/>了解如何使用标记将推送通知从移动服务发送到只有经过身份验证的用户。

+ [将广播通知发送到订阅用户]
	<br/>了解用户如何注册并接收其感兴趣的类别的推送通知。
<!---
+ [将基于模板的通知发送到订阅用户]
	<br/>了解如何使用模板通过移动服务发送推送通知，而无需在后端处理特定于平台的负载。
-->
下面的主题介绍了有关移动服务和通知中心的详细信息：

* [数据处理入门]
  <br/>了解有关使用移动服务存储和查询数据的详细信息。

* [身份验证入门]
  <br/>了解如何通过不同帐户类型（使用移动服务）对应用程序的用户进行身份验证。

* [什么是通知中心？]
  <br/>了解有关通知中心跨所有主要的客户端平台向您的应用程序交付通知的详细信息。

* [调试通知中心应用程序](http://go.microsoft.com/fwlink/p/?linkid=386630)
  </br>获取故障排除和调试通知中心解决方案的指南。 

* [移动服务 Objective-C 操作方法概念性参考]
  <br/>了解有关如何使用移动服务和 Objective C 和 iOS 的详细信息。

* [移动服务服务器脚本参考]
  <br/>了解有关如何在移动服务中实施业务逻辑的详细信息。

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
[安装 Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[iOS 设置门户]: http://go.microsoft.com/fwlink/p/?LinkId=272456
[移动服务 iOS SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[Apple 推送通知服务]: http://go.microsoft.com/fwlink/p/?LinkId=272584
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-ios-get-started
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-ios-get-started-data
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-ios-get-started-users
[Azure 管理门户]: https://manage.windowsazure.cn/
[apns 对象]: http://go.microsoft.com/fwlink/p/?LinkId=272333

[移动服务服务器脚本参考]: /zh-cn/documentation/articles/mobile-services-how-to-use-server-scripts/

[向经过身份验证的用户发送推送通知]: /zh-cn/documentation/articles/mobile-services-javascript-backend-ios-push-notifications-app-users/

[什么是通知中心？]: /zh-cn/documentation/articles/notification-hubs-overview/
[将广播通知发送到订阅用户]: /zh-cn/documentation/articles/notification-hubs-ios-send-breaking-news/
[将基于模板的通知发送到订阅用户]: /zh-cn/documentation/articles/notification-hubs-ios-send-localized-breaking-news/

[移动服务 Objective-C 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library
