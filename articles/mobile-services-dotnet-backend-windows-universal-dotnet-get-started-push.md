<properties
	pageTitle="向通用 Windows 8.1 应用添加推送通知 | Azure"
	description="了解如何从 .NET 后端移动服务使用 Azure 通知中心向通用 Windows 8.1 应用发送推送通知。"
	services="mobile-services,notification-hubs"
	documentationCenter="windows"
	authors="ggailey777"
	manager="erikre"
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="03/05/2016"
	wacn.date="04/18/2016"/>

# 向移动服务应用程序添加推送通知

[AZURE.INCLUDE [mobile-services-selector-get-started-push](../includes/mobile-services-selector-get-started-push.md)]

##概述
本主题说明如何结合使用 Azure 移动服务和 .NET 后端向通用 Windows 应用程序发送推送通知。在本教程中，你将在通用 Windows 应用程序项目中使用 Azure 通知中心来启用推送通知。完成后，每当在 TodoList 表中插入一条记录时，移动服务都会从 .NET 后端向所有已注册的 Windows 应用商店和 Windows Phone 应用商店应用程序发送一条推送通知。创建的通知中心可在移动服务中任意使用，可独立于移动服务进行管理，并可供其他应用程序和服务使用。

若要完成本教程，您需要以下各项：

* 有效的 [Microsoft 应用商店帐户](http://go.microsoft.com/fwlink/p/?LinkId=280045)。
* [Visual Studio Community 2013](https://go.microsoft.com/fwLink/p/?LinkID=391934)。

##<a id="register"></a>为推送通知注册应用程序

[AZURE.INCLUDE [mobile-services-create-new-push-vs2013](../includes/mobile-services-create-new-push-vs2013.md)]

&nbsp;&nbsp;6.浏览到 `\Services\MobileServices\your_service_name` 项目文件夹，打开生成的 push.register.cs 代码文件，并检查 **UploadChannel** 方法是否将设备的通道 URL 注册到通知中心。
 
&nbsp;&nbsp;7.打开共享的 App.xaml.cs 代码文件，可以看到已在 **OnLaunched** 事件处理程序中添加了对新的 **UploadChannel** 方法的调用。这可确保每次启动应用程序时都尝试注册设备。

&nbsp;&nbsp;8.重复执行前面的步骤为 Windows Phone 应用商店应用项目添加推送通知，然后在共享的 App.xaml.cs 文件中，删除对 **UploadChannel** 和剩余 `#if...#endif` 条件包装的额外调用。现在，这两个项目可以共用对 **UploadChannel** 的单一调用。

> [AZURE.NOTE]此外，你还可以通过将 `#if...#endif` 包装的 [MobileServiceClient](http://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobileservices.mobileserviceclient.aspx) 定义统一为单个解包的定义供这两个版本的应用程序使用来简化生成的代码。

现在可以在应用程序中启用推送通知，你必须更新移动服务以发送推送通知。

##<a id="update-service"></a>更新服务以发送推送通知

以下步骤将更新现有的 TodoItemController 类，以便在插入新项时将推送通知发送到所有已注册的设备。你可以在任何自定义的 [ApiController](https://msdn.microsoft.com/zh-cn/library/system.web.http.apicontroller.aspx)、[TableController](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mobile.service.tables.tablecontroller.aspx) 或后端服务中的其他任何位置实现类似的代码。

[AZURE.INCLUDE [mobile-services-dotnet-backend-update-server-push](../includes/mobile-services-dotnet-backend-update-server-push.md)]

##<a id="local-testing"></a>启用推送通知以进行本地测试

[AZURE.INCLUDE [mobile-services-dotnet-backend-configure-local-push-vs2013](../includes/mobile-services-dotnet-backend-configure-local-push-vs2013.md)]

本节中的剩余步骤是可选的。使用这些步骤可以针对本地计算机上运行的移动服务测试你的应用程序。如果你计划使用 Azure 中运行的移动服务测试推送通知，则可以直接跳到最后一节。这是因为“添加推送通知”向导已将你的应用程序配置为连接到 Azure 中运行的服务。

>[AZURE.NOTE]永远不要使用生产用移动服务进行测试和开发工作。始终将你的移动服务项目发布到单独的临时服务进行测试。

&nbsp;&nbsp;5.打开共享的 App.xaml.cs 项目文件，找到创建 [MobileServiceClient] 类的新实例以访问 Azure 中运行的移动服务的代码行。

&nbsp;&nbsp;6.注释掉此代码，并添加创建新的同名但在构造函数中使用本地主机 URL 的 [MobileServiceClient] 的代码，类似于如下代码：

	// This MobileServiceClient has been configured to communicate with your local
	// test project for debugging purposes.
	public static MobileServiceClient todolistClient = new MobileServiceClient(
		"http://localhost:4584"
	);

使用此 [MobileServiceClient]，应用将连接到本地服务（而不是 Azure 中托管的版本）。当你想要切换回来并针对 Azure 中托管的移动服务运行应用程序时，请更改回原始 [MobileServiceClient] 定义。

##<a id="test"></a>在应用程序中测试推送通知

[AZURE.INCLUDE [mobile-services-dotnet-backend-windows-universal-test-push](../includes/mobile-services-dotnet-backend-windows-universal-test-push.md)]

## <a name="next-steps"></a>后续步骤

本教程演示了启用 Windows 应用商店应用程序以便使用移动服务和通知中心发送推送通知的基础知识。建议你接下来完成下一篇教程[向经过身份验证的用户发送推送通知]，其中说明了如何使用标记来做到只将推送通知从移动服务发送到经过身份验证的用户。

通过以下主题了解有关移动服务和通知中心的详细信息：

* [将移动服务添加到现有应用程序][Get started with data]
<br/>了解有关使用移动服务存储和查询数据的详细信息。

* [向应用程序添加身份验证][Get started with authentication]
<br/>了解如何通过不同帐户类型（使用移动服务）对应用程序的用户进行身份验证。

* [什么是通知中心？]
 <br/>了解有关通知中心跨所有主要的客户端平台向你的应用程序交付通知的详细信息。

* [调试通知中心应用程序](http://go.microsoft.com/fwlink/p/?linkid=386630)
</br>获取有关对通知中心解决方案进行故障排除和调试的指导。

* [如何使用适用于 Azure 移动服务的 .NET 客户端]
<br/>了解有关如何从 C# Windows 应用程序使用移动服务的详细信息。

<!-- Anchors. -->

<!-- Images. -->

<!-- URLs. -->
[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[Get started with Mobile Services]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started/
[Get started with data]: /documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-data/
[Get started with authentication]: /documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-users/

[向经过身份验证的用户发送推送通知]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-push-notifications-app-users/

[什么是通知中心？]: /documentation/articles/notification-hubs-overview/

[如何使用适用于 Azure 移动服务的 .NET 客户端]: /documentation/articles/mobile-services-dotnet-how-to-use-client-library/
[MobileServiceClient]: http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mobileservices.mobileserviceclient.aspx

<!---HONumber=Mooncake_0118_2016-->