<properties 
	pageTitle="向 Xamarin Android 应用添加推送通知 | Windows Azure" 
	description="了解如何使用 Azure 移动服务和 Azure 通知中心将推送通知配置为通过 Google Cloud Messaging 发送到 Xamarin.Android 应用程序。" 
	documentationCenter="xamarin" 
	authors="ggailey777" 
	manager="dwrede" 
	services="mobile-services" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="09/16/2015" 
	wacn.date="10/22/2015"/>

#  向移动服务应用程序添加推送通知

[AZURE.INCLUDE [mobile-services-selector-get-started-push](../includes/mobile-services-selector-get-started-push.md)]

## 概述
本主题说明如何使用 Azure 移动服务向 Xamarin.Android 应用程序发送推送通知。在本教程中，你将要使用 Google Cloud Messaging (GCM) 服务向[移动服务入门]项目添加推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。

本教程需要的内容如下：

+ 有效的 Google 帐户
+ [Google Cloud Messaging 客户端组件]。在学习本教程的过程中，你将要添加此组件。

完成[移动服务入门]或[将移动服务添加到现有应用程序]时，你应该已在项目中安装了 [Xamarin.Android] 和 [Azure 移动服务组件]。

## <a id="register"></a>启用 Google Cloud Messaging

[AZURE.INCLUDE [mobile-services-enable-Google-cloud-messaging](../includes/mobile-services-enable-Google-cloud-messaging.md)]

## <a id="configure"></a>配置移动服务以发送推送请求

[AZURE.INCLUDE [mobile-services-android-configure-push](../includes/mobile-services-android-configure-push.md)]

## <a id="update-scripts"></a>更新已注册的插入脚本以发送通知

>[AZURE.TIP]以下步骤说明了如何在 Azure 管理门户中，更新已注册到 TodoItem 表上的插入操作的脚本。你也可以在 Visual Studio 的“服务器资源管理器”的“Azure”节点中直接访问和编辑此移动服务脚本。

[AZURE.INCLUDE [mobile-services-javascript-backend-android-push-insert-script](../includes/mobile-services-javascript-backend-android-push-insert-script.md)]


## <a id="configure-app"></a>为推送通知配置现有项目

[AZURE.INCLUDE [mobile-services-xamarin-android-push-configure-project](../includes/mobile-services-xamarin-android-push-configure-project.md)]

## <a id="add-push"></a>向应用程序添加推送通知代码

[AZURE.INCLUDE [mobile-services-xamarin-android-push-add-to-app](../includes/mobile-services-xamarin-android-push-add-to-app.md)]

## <a id="test"></a>在应用程序中测试推送通知

你可以通过以下方式测试应用程序：使用 USB 电缆直接连接 Android 手机，或者在模拟器中使用虚拟设备。

[AZURE.INCLUDE [mobile-services-android-push-notifications-test](../includes/mobile-services-android-push-notifications-test.md)]

你已成功完成本教程。

##  <a name="next-steps"></a>后续步骤

通过以下主题了解有关移动服务和通知中心的详细信息：

* [将移动服务添加到现有应用程序]
  <br/>了解有关使用移动服务存储和查询数据的详细信息。

* [身份验证入门 ](/documentation/articles/mobile-services-android-get-started-users)
  <br/>了解如何通过移动服务对使用不同帐户类型的应用程序用户进行身份验证。

* [什么是通知中心？](/documentation/articles/notification-hubs-overview)
  <br/>了解有关通知中心跨所有主要的客户端平台向你的应用程序交付通知的详细信息。

* [调试通知中心应用程序](http://go.microsoft.com/fwlink/p/?linkid=386630)
  </br>获取有关对通知中心解决方案进行故障排除和调试的指导。

* [如何使用适用于移动服务的 .NET 客户端库](/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library)
  <br/>了解有关如何将移动服务与 Xamarin C# 代码配合使用的详细信息。

* [移动服务服务器脚本参考](/documentation/articles/mobile-services-how-to-use-server-scripts)
  <br/>了解有关如何在移动服务中实施业务逻辑的详细信息。

<!-- URLs. -->
[移动服务入门]: /documentation/articles/mobile-services-ios-get-started
[将移动服务添加到现有应用程序]: /documentation/articles/mobile-services-android-get-started-data

[Google Cloud Messaging 客户端组件]: http://components.xamarin.com/view/GCMClient/
[Xamarin.Android]: http://xamarin.com/download/
[Azure 移动服务组件]: http://components.xamarin.com/view/azure-mobile-services/
 

<!---HONumber=74-->