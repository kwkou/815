<properties 
	pageTitle="推送通知入门 (Android) | 移动开发人员中心" 
	description="了解如何使用 Azure 移动服务将推送通知发送到 Android .Net 应用程序。" 
	services="mobile-services, notification-hubs" 
	documentationCenter="android" 
	authors="RickSaling" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="07/02/2015" 
	wacn.date="10/03/2015"/>

# 向移动服务应用程序添加推送通知

[AZURE.INCLUDE [mobile-services-selector-get-started-push](../includes/mobile-services-selector-get-started-push.md)]

本主题说明如何使用 Azure 移动服务向 Android 应用程序发送推送通知。在本教程中，你将要使用 Google Cloud Messaging (GCM) 向快速入门项目添加推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。

本教程基于移动服务快速入门。在开始学习本教程之前，必须先完成[移动服务入门]或[数据处理入门]，以将项目连接到移动服务。因此，本教程还需要 Visual Studio 2013。

>[AZURE.NOTE]有关本教程的 Eclipse 版本，请参阅[推送通知入门 (Eclipse)]。

## <a id="register"></a>启用 Google Cloud Messaging

[AZURE.INCLUDE [mobile-services-enable-Google-cloud-messaging](../includes/mobile-services-enable-Google-cloud-messaging.md)]

## <a id="configure"></a>配置移动服务以发送推送请求

[AZURE.INCLUDE [mobile-services-android-configure-push](../includes/mobile-services-android-configure-push.md)]

## <a id="update-server"></a>更新移动服务以发送推送通知

[AZURE.INCLUDE [mobile-services-dotnet-backend-android-push-update-service](../includes/mobile-services-dotnet-backend-android-push-update-service.md)]

## <a name="update-app"></a>向应用程序添加推送通知

### 验证 Android SDK 版本

[AZURE.INCLUDE [mobile-services-verify-android-sdk-version](../includes/mobile-services-verify-android-sdk-version.md)]


下一步就是安装 Google Play Services。Google Cloud Messaging 对开发和测试提出了一些最低的 API 级别要求，清单中的 **minSdkVersion** 属性必须符合这些要求。

如果你要对某台较旧的设备进行测试，请查阅[设置 Google Play Services SDK]，以确定此值可设置到的最小值，并相应地进行设置。

### 将 Google Play Services 添加到项目

[AZURE.INCLUDE [添加 Play Services](../includes/mobile-services-add-Google-play-services.md)]

### 添加代码

[AZURE.INCLUDE [mobile-services-android-getting-started-with-push](../includes/mobile-services-android-getting-started-with-push.md)]

## <a name="test-app"></a>针对发布的移动服务测试应用程序

你可以通过以下方式测试应用程序：使用 USB 电缆直接连接 Android 手机，或者在模拟器中使用虚拟设备。

### <a id="local-testing"></a>启用推送通知以进行本地测试

[AZURE.INCLUDE [mobile-services-dotnet-backend-configure-local-push](../includes/mobile-services-dotnet-backend-configure-local-push.md)]

[AZURE.INCLUDE [mobile-services-android-push-notifications-test](../includes/mobile-services-android-push-notifications-test.md)]

你已成功完成本教程。

## <a name="next-steps"></a>后续步骤

本教程演示了启用 Android 应用程序以便使用移动服务和通知中心发送推送通知的基础知识。建议你接下来完成下一篇教程[向经过身份验证的用户发送推送通知]，其中说明了如何使用标记来做到只将推送通知从移动服务发送到经过身份验证的用户。

+ [向经过身份验证的用户发送推送通知]<br/>了解如何使用标记来做到只将推送通知从移动服务发送到经过身份验证的用户。

+ [将广播通知发送到订户]<br/>了解用户如何注册和接收他们感兴趣的类别的推送通知。

+ [将基于模板的通知发送到订户]<br/>了解如何使用模板从移动服务发送推送通知，且不会在后端中产生平台特定的负载。

通过以下主题了解有关移动服务和通知中心的详细信息：

* [向应用程序添加身份验证][Get started with authentication]<br/>了解如何通过不同帐户类型（使用移动服务）对应用程序的用户进行身份验证。

* [什么是通知中心？] <br/>了解有关通知中心跨所有主要的客户端平台向你的应用程序交付通知的详细信息。

* [调试通知中心应用程序](http://go.microsoft.com/fwlink/p/?linkid=386630)</br>获取有关对通知中心解决方案进行故障排除和调试的指导。

* [如何使用适用于移动服务的 Android 客户端库 ]<br/>了解有关如何将移动服务与 Android 一起使用的详细信息。
  
<!-- Anchors. -->

[Create a new mobile service]: #create-service
[Download the service locally]: #download-the-service-locally
[Test the mobile service]: #test-the-service
[Download the GetStartedWithData project]: #download-app
[Update the app to use the mobile service for data access]: #update-app
[Test the Android App against the service hosted locally]: #test-locally-hosted
[Publish the mobile service to Azure]: #publish-mobile-service
[Test the Android App against the service hosted in Azure]: #test-azure-hosted
[Test the app against the published mobile service]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->

<!-- URLs. -->
[推送通知入门 (Eclipse)]: /documentation/articles/mobile-services-dotnet-backend-android-get-started-push-EC
[移动服务入门]: /documentation/articles/mobile-services-dotnet-backend-android-get-started
[数据处理入门]: /documentation/articles/mobile-services-dotnet-backend-android-get-started-data
[Get started with authentication]: /documentation/articles/mobile-services-dotnet-backend-android-get-started-users
[Management Portal]: https://manage.windowsazure.cn/
[Mobile Services SDK]: http://go.microsoft.com/fwlink/p/?LinkId=257545

[如何使用适用于移动服务的 Android 客户端库 ]: /documentation/articles/mobile-services-android-how-to-use-client-library
[向经过身份验证的用户发送推送通知]: /documentation/articles/mobile-services-dotnet-backend-android-push-notifications-app-users
[什么是通知中心？]: /documentation/articles/notification-hubs-overview
[将广播通知发送到订户]: /documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news
[将基于模板的通知发送到订户]: /documentation/articles/notification-hubs-windows-store-dotnet-send-localized-breaking-news
[Azure Management Portal]: https://manage.windowsazure.cn/

<!---HONumber=71-->