<properties
	pageTitle="推送通知入门 (Android) | 移动开发人员中心
	description="了解如何使用 Azure 移动服务将推送通知发送到 Android .Net 应用程序。"
	services="mobile-services, notification-hubs"
	documentationCenter="android"
	authors="RickSaling"
	manager="dwrede"
	editor=""/>

<tags ms.service="mobile-services" ms.date="02/06/2015" wacn.date="04/15/2015"/>

# 向移动服务应用程序添加推送通知

[AZURE.INCLUDE [mobile-services-selector-get-started-push](../includes/mobile-services-selector-get-started-push-EC.md)]

本主题说明如何使用 Azure 移动服务向 Android 应用程序发送推送通知。在本教程中，你将要使用 Google Cloud Messaging (GCM) 向快速入门项目添加推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。

本教程将指导你完成以下步骤：

1. [启用 Google Cloud Messaging](#register)
2. [配置移动服务以发送推送请求](#configure)
5. [更新服务器以发送推送通知](#update-server)
7. [向应用程序添加推送通知](#update-app)
8. [启用推送通知以进行本地测试](#local-testing)
9. [针对发布的移动服务测试应用程序](#test-app)


本教程基于移动服务快速入门。在开始学习本教程之前，必须先完成[移动服务入门]或[数据处理入门]，以将项目连接到移动服务。因此，本教程还需要 Visual Studio 2013。

>[AZURE.NOTE]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial/)。


## <a id="register"></a>启用 Google Cloud Messaging

[AZURE.INCLUDE [Enable GCM](../includes/mobile-services-enable-Google-cloud-messaging.md)]


## <a id="configure"></a>配置移动服务以发送推送请求

1. 登录到 [Azure 管理门户]、单击"移动服务"，然后单击你的应用。

   	![](./media/mobile-services-android-get-started-push/mobile-services-selection.png)

2. 单击"推送"选项卡，输入你在执行前一过程时从 GCM 获取的"API 密钥"值，然后单击"保存"。

   	![](./media/mobile-services-android-get-started-push/mobile-push-tab-android.png)

> [AZURE.NOTE] 在门户的"推送"选项卡中为增强型推送通知设置 GCM 凭据后，这些凭据将与通知中心共享，以使用你的应用程序配置通知中心。


现在，你的移动服务已配置为使用 GCM 和通知中心。


<h2><a name="download-the-service"></a>将服务下载到本地计算机</h2>

[AZURE.INCLUDE [mobile-services-download-service-locally](../includes/mobile-services-download-service-locally.md)]

<h2><a name="test-the-service"></a>测试移动服务</h2>

[AZURE.INCLUDE [mobile-services-dotnet-backend-test-local-service](../includes/mobile-services-dotnet-backend-test-local-service.md)]

## <a id="update-server"></a>更新服务器以发送推送通知

1. 在 Visual Studio 的解决方案资源管理器中，展开移动服务项目中的 **Controllers** 文件夹。打开 TodoItemController.cs。在该文件的顶部，添加以下 `using` 语句：


		using System;
		using System.Collections.Generic;

2. 使用以下代码更新 `PostTodoItem` 方法定义：  

        public async Task<IHttpActionResult> PostTodoItem(TodoItem item)
        {
            TodoItem current = await InsertAsync(item);

            Dictionary<string, string> data = new Dictionary<string, string>()
            {
                { "message", item.Text}
            };
            GooglePushMessage message = new GooglePushMessage(data, TimeSpan.FromHours(1));

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

    这段代码可在插入 Todo 项之后发送推送通知（包含所插入项的文本）。在发生错误的情况下，这段代码将添加一个错误日志条目，该条目可在管理门户中的移动服务的"日志"选项卡上查看。


<h2><a name="publish-the-service"></a>将移动服务发布到 Azure</h2>

[AZURE.INCLUDE [mobile-services-dotnet-backend-publish-service](../includes/mobile-services-dotnet-backend-publish-service.md)]


## <a name="update-app"></a>向应用程序添加推送通知

### 验证 Android SDK 版本

[AZURE.INCLUDE [mobile-services-verify-android-sdk-version](../includes/mobile-services-verify-android-sdk-version-EC.md)]


下一步就是安装 Google Play Services。Google Cloud Messaging 对开发和测试提出了一些最低的 API 级别要求，清单中的 **minSdkVersion** 属性必须符合这些要求。

如果你要对某台较旧的设备进行测试，请查阅[设置 Google Play Services SDK]，以确定此值可设置到的最小值，并相应地进行设置。

### 将 Google Play Services 添加到项目

[AZURE.INCLUDE [Add Play Services](../includes/mobile-services-add-Google-play-services-EC.md)]

### 添加代码

[AZURE.INCLUDE [mobile-services-android-getting-started-with-push](../includes/mobile-services-android-getting-started-with-push-EC.md)]

<h2><a name="test-app"></a>针对发布的移动服务测试应用程序</h2>

你可以通过以下方式测试应用程序：使用 USB 电缆直接连接 Android 手机，或者在模拟器中使用虚拟设备。

### 如果使用模拟器进行测试...

确保使用支持 Google API 的 Android 虚拟设备 (AVD)。

1. 从"窗口"中选择"Android 虚拟设备管理器"，选择你的设备，然后单击"编辑"（如果你没有任何设备，请单击"新建"）。

	![](./media/mobile-services-android-get-started-push/mobile-services-android-virtual-device-manager.png)

2. 在"目标"中选择"Google API"（或"Google API x86"），然后单击"确定"。

   	![](./media/mobile-services-android-get-started-push/mobile-services-android-virtual-device-manager-edit.png)

	这样便指定了 AVD 要使用 Google API。如果你安装了多个版本的 Android SDK，请确保 API 级别与你前面在项目属性中设置的级别匹配。

### <a id="local-testing"></a> 启用推送通知以进行本地测试

[AZURE.INCLUDE [mobile-services-dotnet-backend-configure-local-push](../includes/mobile-services-dotnet-backend-configure-local-push.md)]

### 运行测试

1. 在 Eclipse 中打开"运行"菜单，然后单击"运行"以启动应用程序。

2. 在应用程序中键入有意义的文本（例如 _A new Mobile Services task_），然后单击"添加"按钮。

  	![](./media/mobile-services-android-get-started-push/mobile-quickstart-push1-android.png)

3. 从屏幕顶部向下轻扫，打开设备的通知中心以查看通知。


你已成功完成本教程。


## <a name="next-steps"> </a>后续步骤

<!---This tutorial demonstrated the basics of enabling an Android app to use Mobile Services and Notification Hubs to send push notifications. Next, consider completing the next tutorial, [Send push notifications to authenticated users], which shows how to use tags to send push notifications from a Mobile Service to only an authenticated user.


+ [Send push notifications to authenticated users]
	<br/>Learn how to use tags to send push notifications from a Mobile Service to only an authenticated user.

+ [Send broadcast notifications to subscribers]
	<br/>Learn how users can register and receive push notifications for categories they're interested in.

+ [Send template-based notifications to subscribers]
	<br/>Learn how to use templates to send push notifications from a Mobile Service, without having to craft platform-specific payloads in your back-end.
-->
通过以下主题了解有关移动服务和通知中心的详细信息：

* [数据处理入门]
  <br/>了解有关使用移动服务存储和查询数据的详细信息。

* [身份验证入门]
  <br/>了解如何通过不同帐户类型（使用移动服务）对应用程序的用户进行身份验证。

* [什么是通知中心？]
  <br/>了解有关通知中心跨所有主要的客户端平台向你的应用程序交付通知的详细信息。

* [调试通知中心应用程序](https://msdn.microsoft.com/zh-cn/library/dn530751.aspx)
  </br>获取故障排除和调试通知中心解决方案的指南。

* [如何使用适用于移动服务的 Android 客户端库]
  <br/>详细了解如何将移动服务与 Android 一起使用。  

<!-- Anchors. -->

[创建新的移动服务]: #create-service
[在本地下载服务]: #download-the-service-locally
[测试移动服务]: #test-the-service
[下载 GetStartedWithData 项目]: #download-app
[更新应用程序以使用移动服务进行数据访问]: #update-app
[针对本地托管的服务测试 Android 应用程序]: #test-locally-hosted
[将移动服务发布到 Azure]: #publish-mobile-service
[针对 Azure 中托管的服务测试 Android 应用程序]: #test-azure-hosted
[针对发布的移动服务测试应用程序]: #test-app
[后续步骤]:#next-steps

<!-- Images. -->
[0]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/app-view.png
[1]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/mobile-data-sample-download-dotnet-vs13.png
[2]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/mobile-service-overview-page.png
[3]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/download-service-project.png
[4]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/add-service-project-to-solution.png
[5]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/download-publishing-profile.png
[6]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/add-existing-project-dialog.png
[7]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/vs-manage-nuget-packages.png
[8]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/manage-nuget-packages.png
[9]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/copy-mobileserviceclient-snippet.png
[10]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/vs-pasted-mobileserviceclient.png
[11]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/vs-build-solution.png
[12]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/vs-run-solution.png
[13]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/new-local-todoitem.png
[14]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/vs-show-local-table-data.png
[15]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/local-item-checked.png
[16]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/azure-items.png
[17]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/manage-sql-azure-database.png
[18]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/sql-azure-query.png

[20]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/vs-build-service-project.png
[21]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/vs-start-debug-service-project.png
[22]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/service-welcome-page.png
[23]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/iis-express-tray.png

[26]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/copy-service-and-packages-folder.png


<!-- URLs. -->
[使用脚本验证和修改数据]: /develop/mobile/tutorials/validate-modify-and-augment-data-dotnet
[使用分页优化查询]: /develop/mobile/tutorials/add-paging-to-data-dotnet
[移动服务入门]: /documentation/articles/mobile-services-dotnet-backend-android-get-started
[数据处理入门]: /documentation/articles/mobile-services-dotnet-backend-android-get-started-data
[身份验证入门]: /documentation/articles/mobile-services-dotnet-backend-android-get-started-users
[JavaScript 和 HTML]: /develop/mobile/tutorials/get-started-with-data-js
[JavaScript 后端版本]: /develop/mobile/tutorials/get-started-with-data-android
[Azure 管理门户]: https://manage.windowsazure.cn/
[管理门户]: https://manage.windowsazure.cn/
[移动服务 SDK]: https://zumo.blob.core.windows.net/sdk/azuresdk-win8-v0.2.5.msi
[开发人员代码示例站点]:  http://code.msdn.microsoft.com/Connect-to-Windows-Azure-3a8fc1ee
[移动服务 .NET 操作方法概念性参考]: /develop/mobile/how-to-guides/work-with-net-client-library
[MobileServiceClient 类]: https://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.windowsazure.mobileservices.mobileserviceclient.aspx

[如何使用适用于移动服务的 Android 客户端库]: /documentation/articles/mobile-services-android-how-to-use-client-library

[向经过身份验证的用户发送推送通知]: /documentation/articles/mobile-services-dotnet-backend-android-push-notifications-app-users/

[什么是通知中心？]: /documentation/articles/notification-hubs-overview/
[将广播通知发送到订户]: /documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news/
[将基于模板的通知发送到订户]: /documentation/articles/notification-hubs-windows-store-dotnet-send-localized-breaking-news/
[Azure 管理门户]: https://manage.windowsazure.cn/

<!--HONumber=50-->
