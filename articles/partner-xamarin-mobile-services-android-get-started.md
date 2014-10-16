<properties pageTitle="Get Started with Mobile Services for Xamarin.Android" metaKeywords="Azure Xamarin.Android application, mobile service XamarinAndroid, getting started Azure Xamarin.Android" writer="craigd" description="Learn how to use Azure Mobile Services with your Xamarin.Android app." metaCanonical="" 
	documentationCenter="Mobile" title="Get started with Mobile Services" />

<a name="getting-started"></a>
# 移动服务入门

<div class="dev-center-tutorial-selector sublanding">
	<a href="/zh-cn/documentation/articles/mobile-services-windows-store-get-started" title="Windows Store">Windows Store</a>
	<a href="/zh-cn/documentation/articles/mobile-services-windows-phone-get-started" title="Windows Phone">Windows Phone</a>
	<a href="/zh-cn/documentation/articles/mobile-services-ios-get-started" title="iOS">iOS</a>
	<a href="/zh-cn/documentation/articles/mobile-services-android-get-started" title="Android">Android</a>
	<a href="/zh-cn/documentation/articles/mobile-services-html-get-started" title="HTML">HTML</a>
	<a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started" title="Xamarin.iOS">Xamarin.iOS</a>
	<a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started" title="Xamarin.Android" class="current">Xamarin.Android</a>
	<a href="/zh-cn/documentation/articles/partner-sencha-mobile-services-get-started/" title="Sencha">Sencha</a>
	<a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-phonegap-get-started/" title="PhoneGap">PhoneGap</a>
</div>

<!--<div class="dev-center-tutorial-subselector">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-xamarin-android-get-started/" title=".NET backend">.NET 后端</a> | 
	<a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started/"  title="JavaScript backend" class="current">JavaScript 后端</a>
</div>-->

<div class="dev-onpage-video-clear clearfix">
<div class="dev-onpage-left-content">
<p>本教程说明如何使用 Azure 移动服务向 Xamarin.Android 应用程序添加基于云的后端服务。在本教程中，你将要创建一个新的移动服务，以及一个在新移动服务中存储应用程序数据的简单*待办事项列表*应用程序。</p>

<p>以下是完成的应用程序的屏幕快照：</p>
</div>

<div class="dev-onpage-video-wrapper"><a href="http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Getting-Started-with-Xamarin-and-Windows-Azure-Mobile-Services" target="_blank" class="label">观看教程</a> <a style="background-image: url('/media/devcenter/mobile/videos/get-started-xamarin-180x120.png') !important;" href="http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Getting-Started-with-Xamarin-and-Windows-Azure-Mobile-Services" target="_blank" class="dev-onpage-video"><span class="icon">播放视频</span></a> <span class="time">10:05</span></div>
</div>

![][0]

完成本教程需要 [Xamarin.Android][1]，该工具将安装 Xamarin Studio 和 Visual Studio 插件（在 Windows 上）以及最新的 Android 平台。需要使用 Android 4.2 SDK 或更高版本。

下载的快速入门项目包含用于 Xamarin.Android 的 Azure 移动服务组件。虽然本项目针对 Android 4.2 或更高版本，但是移动服务 SDK 只需要 Android 2.2 或更高版本。

<div class="dev-callout"><b>说明</b>

<p>若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=A9C9624B5" target="_blank">Azure 免费试用</a>。</p>
</div>

<a name="create-new-service"> </a>
## 创建新的移动服务

[WACOM.INCLUDE [mobile-services-create-new-service](../includes/mobile-services-create-new-service.md)]

## 

## 创建新应用程序创建新的 Xamarin.Android 应用程序

创建移动服务后，你可以在管理门户中遵照一个简易的快速入门项目来创建新应用程序或修改现有应用程序，以连接到你的移动服务。

在本部分中，你将要创建一个连接到移动服务的新的 Xamarin.Android 应用程序。

1.  在管理门户中单击“移动服务”，然后单击你刚刚创建的移动服务 。

2.  在快速入门选项卡中，单击“选择平台”下的“Xamarin.Android”，然后展开“创建新的 Android 应用程序” 。

    ![][2]

    此时将显示三个简单步骤，描述如何创建与移动服务连接的 Xamarin.Android 应用程序。

    ![][3]

3.  单击“创建 TodoItem 表” 以创建用于存储应用程序数据的表。

4.  在“下载并运行应用程序” 下面单击“下载” 。

    随即将会下载已连接到移动服务的示例*待办事项列表*应用程序的项目。将压缩的项目文件保存到本地计算机，并记下保存位置。

## 运行 Android 应用程序

本教程的最后一个阶段是生成和运行你的新应用程序。

1.  浏览到压缩的项目文件所保存到的位置，然后在计算机上展开这些文件。

2.  在 Xamarin Studio 或 Visual Studio 中，依次单击“文件” 、“打开” ，导航到解压缩的示例文件，然后选择“XamarinTodoQuickStart.Android.sln” 以将其打开。

    ![][4]

    ![][5]

3.  按“运行” 按钮生成项目并启动应用程序。系统将要求你选择模拟器或已连接的 USB 设备。

    "说明"

    若要在 Android 模拟器中运行项目，必须至少定义一个 Android 虚拟设备 (AVD)。使用 AVD 管理器创建和管理这些设备。

4.  在应用程序中键入有意义的文本（例如 *Complete the tutorial*），然后单击“添加” 。

    ![][6]

    这样可向在 Azure 中托管的新移动服务发送 POST 请求。来自请求的数据被插入到 TodoItem 表。移动服务返回存储在表中的项，数据显示在列表中。

    > [WACOM.NOTE]
    > 你可以查看访问你的移动服务以查询和插入数据的代码，这些代码在 ToDoActivity.cs C\# 文件中。

5.  返回管理门户，单击“数据” 选项卡，然后单击“TodoItems” 表。

    ![][7]

    此时，你可以浏览应用程序在表中插入的数据。

    ![][8]

<a name="next-steps"> </a>
## 后续步骤

完成快速入门后，请了解如何在移动服务中执行其他重要任务：

-   [数据处理入门][]
    了解有关使用移动服务存储和查询数据的详细信息。

-   [身份验证入门][]
    了解如何使用标识提供者对应用程序的用户进行身份验证。

-   [推送通知入门][]
    了解如何向应用程序发送一条非常简单的推送通知。

  [Windows 应用商店]: /zh-cn/documentation/articles/mobile-services-windows-store-get-started "Windows 应用商店"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/mobile-services-ios-get-started "iOS"
  [Android]: /zh-cn/documentation/articles/mobile-services-android-get-started "Android"
  [HTML]: /zh-cn/documentation/articles/mobile-services-html-get-started "HTML"
  [Xamarin.iOS]: /zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started "Xamarin.Android"
  [Sencha]: /zh-cn/documentation/articles/partner-sencha-mobile-services-get-started/ "Sencha"
  [PhoneGap]: /zh-cn/documentation/articles/mobile-services-javascript-backend-phonegap-get-started/ "PhoneGap"
  [观看教程]: http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Getting-Started-with-Xamarin-and-Windows-Azure-Mobile-Services
  [0]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-quickstart-completed-android.png
  [1]: http://xamarin.com/download
  [Azure 免费试用]: http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=A9C9624B5
  [mobile-services-create-new-service]: ../includes/mobile-services-create-new-service.md
  [2]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-portal-quickstart-xamarin-android.png
  [3]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-quickstart-steps-xamarin-android.png
  [4]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-xamarin-project-android-xs.png
  [5]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-xamarin-project-android-vs.png
  [6]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-quickstart-startup-android.png
  [7]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-data-tab.png
  [8]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-data-browse.png
  [数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-android
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-android
  [推送通知入门]: /zh-cn/develop/mobile/tutorials/get-started-with-push-xamarin-android
