<properties 
	pageTitle="向通用 Windows 8.1 应用添加推送通知 | Microsoft Azure" 
	description="了解如何从 JavaScript 后端移动服务使用 Azure 通知中心向通用 Windows 8.1 应用发送推送通知。" 
	services="mobile-services,notification-hubs" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="03/05/2016" 
	wacn.date="04/11/2016"/>


#  向移动服务应用程序添加推送通知

[AZURE.INCLUDE [mobile-services-selector-get-started-push](../includes/mobile-services-selector-get-started-push.md)]

 
本主题说明如何结合使用 Azure 移动服务和 JavaScript 后端向通用 Windows 应用程序发送推送通知。在本教程中，你将在通用 Windows 应用程序项目中使用 Azure 通知中心来启用推送通知。完成后，当每次在 TodoList 表中插入一条记录时，你的移动服务将从 JavaScript 后端向所有已注册的 Windows 应用商店和 Windows Phone 应用商店应用发送一条推送通知。创建的通知中心可在移动服务中任意使用，可独立于移动服务进行管理，并可供其他应用程序和服务使用。

>[AZURE.NOTE]本主题演示如何在 Visual Studio 2013 Update 3 中使用此工具，以便从移动服务向通用 Windows 应用程序添加推送通知的支持。可以使用相同的步骤将推送通知从移动服务添加到 Windows 应用商店或 Windows Phone 应用商店 8.1 应用程序。若要将推送通知添加到 Windows Phone 8 或 Windows Phone Silverlight 8.1 应用程序，请参阅此版本的[移动服务中的推送通知入门](/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push/)。

本教程将指导你完成启用推送通知的以下基本步骤：

1. [为推送通知注册应用程序](#register)
2. [更新服务以发送推送通知](#update-service)
3. [在应用程序中测试推送通知](#test)

若要完成本教程，您需要以下各项：

* 有效的 [Microsoft 应用商店帐户](http://go.microsoft.com/fwlink/p/?LinkId=280045)。
* [Visual Studio 2013 Express for Windows](http://go.microsoft.com/fwlink/?LinkId=257546) Update 3 或更高版本

## <a id="register"></a>为推送通知注册应用程序

[AZURE.INCLUDE [mobile-services-create-new-push-vs2013](../includes/mobile-services-create-new-push-vs2013.md)]

&nbsp;&nbsp;6.浏览到 `\Services\MobileServices\your_service_name` 项目文件夹，打开生成的 push.register.cs 代码文件，并检查 **UploadChannel** 方法是否将设备的通道 URL 注册到通知中心。

&nbsp;&nbsp;7.打开共享的 App.xaml.cs 代码文件，可以看到已在 **OnLaunched** 事件处理程序中添加了对新的 **UploadChannel** 方法的调用。这可确保每次启动应用程序时都尝试注册设备。

&nbsp;&nbsp;8.重复执行前面的步骤为 Windows Phone 应用商店应用项目添加推送通知，然后在共享的 App.xaml.cs 文件中，删除对**移动服务客户端**、**UploadChannel** 和剩余 `#if...#endif` 条件包装的额外调用。现在，这两个项目可以共用对 **UploadChannel** 的单一调用。

&nbsp;&nbsp;请注意，你还可以通过将 `#if...#endif` 包装的 [MobileServiceClient] 定义统一为单个解包的定义，供这两个版本的应用使用来简化生成的代码。

现在可以在应用程序中启用推送通知，你必须更新移动服务以发送推送通知。

## <a id="update-service"></a>更新服务以发送推送通知

以下步骤更新注册到 TodoItem 表的插入脚本。在任何服务器脚本中或在后端服务中的其他任何地方，您都可以实施类似的代码。

[AZURE.INCLUDE [mobile-services-javascript-update-script-notification-hubs](../includes/mobile-services-javascript-update-script-notification-hubs.md)]


## <a id="test"></a>在应用程序中测试推送通知

[AZURE.INCLUDE [mobile-services-javascript-backend-windows-universal-test-push](../includes/mobile-services-javascript-backend-windows-universal-test-push.md)]

##  <a name="next-steps"></a>后续步骤

本教程演示了启用 Windows 应用商店应用程序以便使用移动服务和通知中心发送推送通知的基础知识。接下来，请考虑完成以下教程之一：

+ [向经过身份验证的用户发送推送通知](/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-push-notifications-app-users/)
<br/>了解如何使用标记来做到只将推送通知从移动服务发送到经过身份验证的用户。

+ [将广播通知发送到订户](/documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news/)
<br/>了解用户如何注册和接收他们感兴趣的类别的推送通知。

+ [将平台无关的通知发送到订户](/documentation/articles/notification-hubs-aspnet-cross-platform-notify-users/)
<br/>了解如何使用模板从移动服务发送推送通知，且不会在后端中产生平台特定的负载。

	通过以下主题了解有关移动服务和通知中心的详细信息：

* [Azure 通知中心 - 诊断指南](/documentation/articles/notification-hubs-diagnosing/)
<br/>了解如何排查推送通知问题。

* [身份验证入门]
  <br/>了解如何通过移动服务对使用不同帐户类型的应用程序用户进行身份验证。


* [什么是通知中心？] 
<br/>了解有关通知中心跨所有主要的客户端平台向你的应用程序交付通知的详细信息。

* [如何使用适用于 Azure 移动服务的 .NET 客户端]
<br/>了解有关如何从 C# Windows 应用程序使用移动服务的详细信息。

<!-- Anchors. -->

<!-- Images. -->

<!-- URLs. -->

[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[Get started with Mobile Services]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started/
[身份验证入门]: /documentation/articles/mobile-services-javascript-backend-windows-universal-dotnet-get-started-users/

[Send push notifications to authenticated users]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-push-notifications-app-users/

[什么是通知中心？]: /documentation/articles/notification-hubs-overview/

[如何使用适用于 Azure 移动服务的 .NET 客户端]: /documentation/articles/mobile-services-dotnet-how-to-use-client-library/
[MobileServiceClient]: http://go.microsoft.com/fwlink/p/?LinkId=302030

<!---HONumber=Mooncake_1221_2015-->