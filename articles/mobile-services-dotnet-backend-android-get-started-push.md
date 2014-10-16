<properties linkid="develop-mobile-tutorials-dotnet-backend-get-started-with-push-android" urlDisplayName="Get Started with Push" pageTitle="Get started with push (Android) | Mobile Dev Center" metaKeywords="" description="Learn how to use Azure Mobile Services to send push notifications to your Android .Net app." metaCanonical="" services="" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services" authors="ricksal" solutions="" manager="dwrede" editor="" />

# 移动服务中的推送通知入门

<div class="dev-center-tutorial-selector sublanding">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-push/" title="Windows Store C#">Windows 应用商店 C\#</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-push/" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-push/" title="Windows Phone">Windows Phone</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started-push/" title="Android" class="current">Android</a>

</div>

<div class="dev-center-tutorial-subselector">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started-push/" title=".NET backend" class="current">.NET 后端</a> | 
	<a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-android-get-started-push/"  title="JavaScript backend">JavaScript 后端</a>
</div>

本主题说明如何使用 Azure 移动服务向 Android 应用程序发送推送通知。在本教程中，你将要使用 Google Cloud Messaging (GCM) 向快速入门项目添加推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。

在本教程中创建的移动服务支持移动服务中的 .NET 运行时。这样，你便可以将 .NET 语言和 Visual Studio 用于移动服务中的服务器端业务逻辑。若要创建允许以 JavaScript 编写服务器端业务逻辑的移动服务，请参阅本主题中的 [JavaScript 后端版本][]。

[WACOM.NOTE] 本教程演示了移动服务与通知中心的集成功能，该功能当前以预览版提供。

<div class="dev-callout"><b>说明</b>

<p>本教程需要安装 Visual Studio 2013。</p>
</div>

本教程将指导你完成以下基本步骤：

1.  [启用 Google Cloud Messaging][]
2.  [配置移动服务以发送推送请求][]
3.  [在本地下载服务][]
4.  [测试移动服务][]
5.  [更新服务器以发送推送通知][]
6.  [将移动服务发布到 Azure][]
7.  [向应用程序添加推送通知][]
8.  [针对发布的移动服务测试应用程序][]

本教程基于移动服务快速入门。在开始学习本教程之前，必须先完成[移动服务入门][]或[数据处理入门][]，以将项目连接到移动服务。

<div class="dev-callout"><b>说明</b>

若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=AE564AB28&amp;returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fzh-cn%2Fdocumentation%2Farticles%2Fmobile-services-dotnet-backend-windows-store-dotnet-get-started-data%2F" target="_blank">Azure 免费试用</a>。<p>
</div>

<a id="register"></a>
## 启用 Google Cloud Messaging

[WACOM.INCLUDE [启用 GCM][]]

<a id="configure"></a>
## 配置移动服务以发送推送请求

1.  登录到 [Windows Azure 管理门户]，单击“移动服务”，然后单击你的应用程序 。

    ![][]

2.  单击“推送” 选项卡，输入你在执行前一过程时从 GCM 获取的“API 密钥” 值，然后单击“保存” 。

    ![][1]

    <div class="dev-callout"><b>重要说明</b>

    <p>在门户的“推送”选项卡中为增强型推送通知设置 GCM 凭据后，这些凭据将与通知中心共享，以使用你的应用程序配置通知中心。</p>
	</div>

现在，你的移动服务已配置为使用 GCM 和通知中心。

<a name="download-the-service"></a>
## 下载服务将服务下载到本地计算机

[WACOM.INCLUDE [mobile-services-download-service-locally][]]

<a name="test-the-service"></a>
## 测试服务测试移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-test-local-service][]]

<a name="publish-the-service"></a>
## 更新服务器以发送推送通知

[WACOM.INCLUDE [mobile-services-dotnet-backend-update-server-GCM][]]

<a name="update-app"></a>
## 发布服务将移动服务发布到 Azure

[WACOM.INCLUDE [mobile-services-dotnet-backend-publish-service][]]

## 向应用程序添加推送通知

### 验证 Android SDK 版本

[WACOM.INCLUDE [mobile-services-verify-android-sdk-version][]]

下一步就是安装 Google Play Services。Google Cloud Messaging 对开发和测试提出了一些最低的 API 级别要求，清单中的 "minSdkVersion" 属性必须符合这些要求。

如果你要对某台较旧的设备进行测试，请查阅 [设置 Google Play Services SDK]，以确定此值可设置到的最小值，并相应地进行设置。

### 将 Google Play Services 添加到项目

[WACOM.INCLUDE [添加 Play Services][]]

### 添加代码

[WACOM.INCLUDE [mobile-services-android-getting-started-with-push][]]

## 测试应用程序针对发布的移动服务测试应用程序

你可以通过以下方式测试应用程序：使用 USB 电缆直接连接 Android 手机，或者在模拟器中使用虚拟设备。

### 如果使用模拟器进行测试...

确保使用支持 Google API 的 Android 虚拟设备 (AVD)。

1.  从“窗口”中选择“Android 虚拟设备管理器”，选择你的设备，然后单击“编辑”（如果你没有任何设备，请单击“新建”） 。

    ![][2]

2.  在“目标” 中选择“Google API” （或“Google API x86”） ，然后单击“确定”。

    ![][3]

    这样便指定了 AVD 要使用 Google API。如果你安装了多个版本的 Android SDK，请确保 API 级别与你前面在项目属性中设置的级别匹配。

### 运行测试

1.  在 Eclipse 中打开“运行” 菜单，然后单击“运行”以启动应用程序 。

2.  在应用程序中键入有意义的文本（例如 *A new Mobile Services task*），然后单击“添加” 按钮。

    ![][4]

3.  从屏幕顶部向下轻扫，打开设备的通知中心以查看通知。

你已成功完成本教程。

<a name="next-steps"> </a>
## 后续步骤

本教程演示了移动服务提供的基本推送通知功能。如果你的应用程序需要更高级功能，例如发送跨平台通知、基于订阅的路由或极大量的通知，请考虑为移动服务使用 Windows Azure 通知中心。有关详细信息，请参阅下列通知中心主题之一：

-   [通知中心入门][]
    了解如何在 Android 应用程序中利用通知中心。

-   [向订户发送通知][]
    了解用户如何注册和接收他们感兴趣的类别的推送通知。

-   [向用户发送通知][]
    了解如何从移动服务向任一设备上的特定用户发送推送通知。

-   [向用户发送跨平台通知][]
    了解如何使用模板从移动服务发送推送通知，且不会在后端中产生平台特定的负载。

建议你了解有关以下移动服务主题的详细信息：

-   [数据处理入门][]
    了解有关使用移动服务存储和查询数据的详细信息。

-   [身份验证入门][]
    了解如何使用 Windows 帐户对应用程序用户进行身份验证。

-   [移动服务 .NET 操作方法概念性参考][]
    了解有关如何将移动服务与 .NET 一起使用的详细信息。

-   [如何使用适用于移动服务的 Android 客户端库][]
    了解有关如何将移动服务与 Android 一起使用的详细信息。

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-push/ "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-push/ "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-push/ "Windows Phone"
  [Android]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started-push/ "Android"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started-push/ ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-javascript-backend-android-get-started-push/ "JavaScript 后端"
  [JavaScript 后端版本]: /zh-cn/develop/mobile/tutorials/get-started-with-data-android
  [启用 Google Cloud Messaging]: #register
  [配置移动服务以发送推送请求]: #configure
  [在本地下载服务]: #download-the-service-locally
  [测试移动服务]: #test-the-service
  [更新服务器以发送推送通知]: #update-server
  [将移动服务发布到 Azure]: #publish-mobile-service
  [向应用程序添加推送通知]: #update
  [针对发布的移动服务测试应用程序]: #test-app
  [移动服务入门]: /zh-cn/documentation/articles/mobile-services-android-get-started/
  [数据处理入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started-data/
  [Azure 免费试用]: http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=AE564AB28&returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fzh-cn%2Fdocumentation%2Farticles%2Fmobile-services-dotnet-backend-windows-store-dotnet-get-started-data%2F
  [启用 GCM]: ../includes/mobile-services-enable-Google-cloud-messaging.md
  [0]: ./media/mobile-services-android-get-started-push/mobile-services-selection.png
  [1]: ./media/mobile-services-android-get-started-push/mobile-push-tab-android.png
  [mobile-services-download-service-locally]: ../includes/mobile-services-download-service-locally.md
  [mobile-services-dotnet-backend-test-local-service]: ../includes/mobile-services-dotnet-backend-test-local-service.md
  [mobile-services-dotnet-backend-update-server-GCM]: ../includes/mobile-services-dotnet-backend-update-server-GCM.md
  [mobile-services-dotnet-backend-publish-service]: ../includes/mobile-services-dotnet-backend-publish-service.md
  [mobile-services-verify-android-sdk-version]: ../includes/mobile-services-verify-android-sdk-version.md
  [添加 Play Services]: ../includes/mobile-services-add-Google-play-services.md
  [mobile-services-android-getting-started-with-push]: ../includes/mobile-services-android-getting-started-with-push.md
  [2]: ./media/mobile-services-android-get-started-push/mobile-services-android-virtual-device-manager.png
  [3]: ./media/mobile-services-android-get-started-push/mobile-services-android-virtual-device-manager-edit.png
  [4]: ./media/mobile-services-android-get-started-push/mobile-quickstart-push1-android.png
  [通知中心入门]: /zh-cn/manage/services/notification-hubs/getting-started-windows-dotnet/
  [向订户发送通知]: /zh-cn/manage/services/notification-hubs/breaking-news-dotnet/
  [向用户发送通知]: /zh-cn/manage/services/notification-hubs/notify-users/
  [向用户发送跨平台通知]: /zh-cn/manage/services/notification-hubs/notify-users-xplat-mobile-services/
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-android
  [移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library
  [如何使用适用于移动服务的 Android 客户端库]: /zh-cn/documentation/articles/mobile-services-android-how-to-use-client-library
