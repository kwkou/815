<properties
	pageTitle="向应用程序添加推送通知 (iOS) | .NET 后端"
	description="了解如何使用 Azure 移动服务将推送通知发送到 iOS 应用程序。"
	services="mobile-services,notification-hubs"
	documentationCenter="ios"
	manager="dwrede"
	editor=""
	authors="krisragh"/>

<tags
	ms.service="mobile-services"
	ms.date="03/09/2016"
	wacn.date="04/18/2016"/>


# 向 iOS 应用和 .NET 后端添加推送通知

[AZURE.INCLUDE [mobile-services-selector-get-started-push](../includes/mobile-services-selector-get-started-push.md)]

本教程说明将推送通知发送到[快速入门项目](/documentation/articles/mobile-services-dotnet-backend-ios-get-started/)，这样，每次插入一条记录时，你的移动服务就会发送一条推送通知。你必须先完成[移动服务入门]教程。

[AZURE.INCLUDE [启用 Apple 推送通知](../includes/enable-apple-push-notifications.md)]

## <a id="configure"></a>配置 Azure 以发送推送通知

[AZURE.INCLUDE [在 Azure 移动服务中配置推送通知](../includes/mobile-services-apns-configure-push.md)]

##<a id="update-server"></a>更新后端代码以发送推送通知

* 打开 Visual Studio 项目，然后选择“控制器”文件夹 >“TodoItemController.cs”> 方法 `PostTodoItem`。将该方法替换为以下内容。插入待办事项时，此代码将发送包含项文本的推送通知。如果发生了错误，该代码将添加一个错误日志条目，该条目可通过门户中的“日志”部分查看。


```
        public async Task<IHttpActionResult> PostTodoItem(TodoItem item)
        {
            TodoItem current = await InsertAsync(item);

            ApplePushMessage message = new ApplePushMessage(item.Text, System.TimeSpan.FromHours(1));

            try
            {
                var result = await Services.Push.SendAsync(message);
                Services.Log.Info(result.State.ToString());
            }
            catch (System.Exception ex)
            {
                Services.Log.Error(ex.Message, null, "Push.SendAsync Error");
            }
            return CreatedAtRoute("Tables", new { id = current.Id }, current);
        }
```

##<a name="publish-the-service"></a>将移动服务发布到 Azure

[AZURE.INCLUDE [mobile-services-dotnet-backend-publish-service](../includes/mobile-services-dotnet-backend-publish-service.md)]

[AZURE.INCLUDE [向应用程序添加推送通知](../includes/add-push-notifications-to-app.md)]

[AZURE.INCLUDE [在应用程序中测试推送通知](../includes/test-push-notifications-in-app.md)]

<!-- Anchors.  -->
[Generate the certificate signing request]: #certificates
[Register your app and enable push notifications]: #register
[Create a provisioning profile for the app]: #profile
[Configure Mobile Services]: #configure
[Update scripts to send push notifications]: #update-scripts
[Add push notifications to the app]: #add-push
[Insert data to receive notifications]: #test
[Test the app against the published mobile service]: #test-app
[Next Steps]: #next-steps
[Download the service locally]: #download-the-service-locally
[Test the mobile service]: #test-the-service
[Publish the mobile service to Azure]: #publish-mobile-service

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

<!-- URLs. -->
[Install Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[iOS Provisioning Portal]: http://go.microsoft.com/fwlink/p/?LinkId=272456
[Mobile Services iOS SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[Apple Push Notification Service]: http://go.microsoft.com/fwlink/p/?LinkId=272584
[移动服务入门]: /documentation/articles/mobile-services-dotnet-backend-ios-get-started/
[apns object]: http://go.microsoft.com/fwlink/p/?LinkId=272333

[Get started with authentication]: /documentation/articles/mobile-services-dotnet-backend-ios-get-started-users/
[Mobile Services Objective-C how-to conceptual reference]: /documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library/
[What are Notification Hubs?]: /documentation/articles/notification-hubs-overview/
[Send broadcast notifications to subscribers]: /documentation/articles/notification-hubs-ios-send-breaking-news/
[Send template-based notifications to subscribers]: /documentation/articles/notification-hubs-ios-send-localized-breaking-news/

<!---HONumber=Mooncake_0215_2016-->