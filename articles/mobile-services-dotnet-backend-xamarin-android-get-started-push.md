<properties 
	pageTitle="用于 Xamarin Android 应用程序的移动服务入门 - Azure 移动服务" 
	description="了解如何使用 Azure 移动服务和通知中心将推送通知发送到 Xamarin Android 应用程序" 
	services="mobile-services" 
	documentationCenter="xamarin" 
	authors="ggailey777" 
	manager="dwrede" 
	editor="mollybos"/>

<tags 
	ms.service="mobile-services" 
	ms.date="08/18/2015" 
	wacn.date="10/03/2015"/>

# 向移动服务应用程序添加推送通知

[AZURE.INCLUDE [mobile-services-selector-get-started-push](../includes/mobile-services-selector-get-started-push.md)]

本主题说明如何使用 Azure 移动服务向 Xamarin.Android 应用程序发送推送通知。在本教程中，你将要使用 Google Cloud Messaging (GCM) 服务向[移动服务入门]项目添加推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。

本教程将指导你完成启用推送通知的以下基本步骤：

1. [启用 Google Cloud Messaging](#register)
2. [配置移动服务](#configure)
3. [为推送通知配置项目](#configure-app)
4. [向应用程序添加推送通知代码](#add-push)
5. [插入数据以接收通知](#test)

本教程需要的内容如下：

+ 有效的 Google 帐户
+ [Google Cloud Messaging 客户端组件]。在学习本教程的过程中，你将要添加此组件。

完成[移动服务入门]教程时，你应该已在项目中安装了 [Xamarin.Android] 和 [Azure 移动服务][Azure Mobile Services Component]组件。

##<a id="register"></a>启用 Google Cloud Messaging

[AZURE.INCLUDE [启用 GCM](../includes/mobile-services-enable-Google-cloud-messaging.md)]

##<a id="configure"></a>配置移动服务以发送推送请求

[AZURE.INCLUDE [mobile-services-android-configure-push](../includes/mobile-services-android-configure-push.md)]

##<a id="update-server"></a>更新移动服务以发送推送通知

[AZURE.INCLUDE [mobile-services-dotnet-backend-android-push-update-service](../includes/mobile-services-dotnet-backend-android-push-update-service.md)]

##<a id="configure-app"></a>为推送通知配置现有项目

[AZURE.INCLUDE [mobile-services-xamarin-android-push-configure-project](../includes/mobile-services-xamarin-android-push-configure-project.md)]

##<a id="add-push"></a>向应用程序添加推送通知代码

[AZURE.INCLUDE [mobile-services-xamarin-android-push-add-to-app](../includes/mobile-services-xamarin-android-push-add-to-app.md)]

##<a name="test-app"></a>针对发布的移动服务测试应用程序

你可以通过以下方式测试应用程序：使用 USB 电缆直接连接 Android 手机，或者在模拟器中使用虚拟设备。

###<a id="local-testing"></a>启用推送通知以进行本地测试

[AZURE.INCLUDE [mobile-services-dotnet-backend-configure-local-push](../includes/mobile-services-dotnet-backend-configure-local-push.md)]

[AZURE.INCLUDE [mobile-services-android-push-notifications-test](../includes/mobile-services-android-push-notifications-test.md)]

<!-- URLs. -->
[移动服务入门]: /documentation/articles/mobile-services-dotnet-backend-xamarin-android-get-started
[Google Cloud Messaging 客户端组件]: http://components.xamarin.com/view/GCMClient/
[Xamarin.Android]: http://xamarin.com/download/
[Azure Mobile Services Component]: http://components.xamarin.com/view/azure-mobile-services/

<!---HONumber=71-->