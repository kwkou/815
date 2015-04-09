<properties pageTitle="用于 iOS 应用程序的 Azure 移动服务入门" metaKeywords="Azure iOS application, mobile service iOS, getting started Azure iOS" description="按照本教程进行操作，开始使用 Azure 移动服务进行 iOS 开发。 " metaCanonical="" services="" documentationCenter="Mobile" title="Get started with Mobile Services" authors="glenga" solutions="" manager="" editor="" />

# <a name="getting-started"> </a>移动服务入门

[WACOM.INCLUDE [mobile-services-selector-get-started](../includes/mobile-services-selector-get-started.md)]

<div class="dev-onpage-video-clear clearfix">
<div class="dev-onpage-left-content">
<p>本教程说明如何使用 Azure 移动服务向 iOS 应用程序添加基于云的后端服务。</p>
<p>如果你更愿意观看视频，右侧的视频片段提供了与本教程相同的步骤。</p>
</div>
<div class="dev-onpage-video-wrapper" style="display:none"><a href="http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/iOS-Creating-your-first-app-using-the-Windows-Azure-Mobile-Services-Quickstart" target="_blank" class="label">观看教程</a> <a style="background-image: url('/media/devcenter/mobile/videos/get-started-ios-180x120.png') !important;" href="http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/iOS-Creating-your-first-app-using-the-Windows-Azure-Mobile-Services-Quickstart" target="_blank" class="dev-onpage-video"><span class="icon">播放视频</span></a> <span class="time">9:38</span></div>
</div>

在本教程中，你将要创建一个新的移动服务，以及一个在新移动服务中存储应用程序数据的简单待办事项列表应用程序。要创建的移动服务将为服务器端业务逻辑使用 JavaScript。若要创建允许你使用 Visual Studio 以受支持 .NET 语言编写服务器端业务逻辑的移动服务，请参阅本主题的 [.NET 后端版本]。

以下是完成的应用程序的屏幕截图：

![][0]

完成本教程需要安装 XCode 4.5 和 iOS 5.0 或更高版本。

> [AZURE.NOTE] 若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以注册 Azure 试用版并获取最多 10 个免费移动服务，这些移动服务即使在你的试用期结束后也可以继续使用。有关详细信息，请参阅 [Azure 免费试用](/zh-cn/pricing/1rmb-trial/?WT.mc_id=AE564AB28&amp;returnurl=http%3A%2F%2Fwww.windowsazure.cn%2Fzh-cn%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started-ios%2F%20 target="_blank")。

## <a name="create-new-service"> </a>创建新的移动服务

[WACOM.INCLUDE [mobile-services-create-new-service](../includes/mobile-services-create-new-service.md)]

<h2>创建新的 iOS 应用程序</h2>

创建移动服务后，你可以在管理门户中遵照一个简易的快速入门项目来创建新应用程序或修改现有应用程序，以连接到你的移动服务。

在本部分中，你将要创建一个连接到移动服务的新的 iOS 应用程序。

1.  在管理门户中单击"移动服务"，然后单击您刚刚创建的移动服务。

2. 在快速入门选项卡中，单击"选择平台"下的**iOS**，然后展开"创建新的 iOS 应用程序"。

   	![][6]

   	此时将显示三个简单步骤，描述如何创建与移动服务连接的 iOS 应用程序。

  	![][7]

3. 下载并安装 [Xcode] v4.4 或更高版本（如果尚未这么做）。

4. 单击"创建 **TodoItems** 表"以创建用于存储应用程序数据的表。

5. 在**"下载并运行应用程序"**下面单击**"下载"**。 

  	随即将会下载已连接到移动服务的示例_待办事项列表_应用程序的项目，以及移动服务 iOS SDK。将压缩的项目文件保存到本地计算机，并记下保存位置。

## 运行新的 iOS 应用程序

[WACOM.INCLUDE [mobile-services-ios-run-app](../includes/mobile-services-ios-run-app.md)]

<ol start="4">
<li><p>返回到管理门户中，单击 <strong>"数据"</strong> 选项卡，然后单击 <strong>TodoItems</strong> 表。<p></li></ol>

![](./media/mobile-services-ios-get-started/mobile-data-tab.png)

此时，你可以浏览应用程序在表中插入的数据。</p>

![](./media/mobile-services-ios-get-started/mobile-data-browse.png)


## <a name="next-steps"> </a>后续步骤
完成快速入门后，请了解如何在移动服务中执行其他重要任务：

* [数据处理入门]
  <br/>了解有关使用移动服务存储和查询数据的详细信息。

* [脱机数据同步入门]
  <br/>了解如何使用脱机数据同步让应用程序响应迅速且可靠。

* [身份验证入门]
  <br/>了解如何使用标识提供程序对应用程序的用户进行身份验证。

* [推送通知入门]
  <br/>了解如何将非常简单的推送通知发送到你的应用程序。


<!-- Anchors. -->
[移动服务入门]:#getting-started
[创建新的移动服务]:#create-new-service
[定义移动服务实例]:#define-mobile-service-instance
[后续步骤]:#next-steps

<!-- Images. -->
[0]: ./media/mobile-services-ios-get-started/mobile-quickstart-completed-ios.png

[6]: ./media/mobile-services-ios-get-started/mobile-portal-quickstart-ios.png
[7]: ./media/mobile-services-ios-get-started/mobile-quickstart-steps-ios.png
[8]: ./media/mobile-services-ios-get-started/mobile-xcode-project.png

[10]: ./media/mobile-services-ios-get-started/mobile-quickstart-startup-ios.png
[11]: ./media/mobile-services-ios-get-started/mobile-data-tab.png
[12]: ./media/mobile-services-ios-get-started/mobile-data-browse.png


<!-- URLs. -->
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-data-ios
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-users-ios
[脱机数据同步入门]: /zh-cn/documentation/articles/mobile-services-ios-get-started-offline-data
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-ios

[移动服务 iOS SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[管理门户]: https://manage.windowsazure.cn/
[XCode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[.NET 后端版本]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-get-started
