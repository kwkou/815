<properties pageTitle="Get Started with Mobile Services for Windows Store apps | Mobile Dev Center" metaKeywords="" description="Follow this tutorial to get started using Azure Mobile Services for Windows Store development in C# or JavaScript. " metaCanonical="" services="" documentationCenter="Mobile" title="Get started with Mobile Services" authors="glenga" solutions="" manager="" editor="" />

<a name="getting-started"> </a>
# 移动服务入门

<div class="dev-center-tutorial-selector sublanding">
	<a href="/zh-cn/documentation/articles/mobile-services-windows-store-get-started" title="Windows Store" class="current">Windows Store</a>
	<a href="/zh-cn/documentation/articles/mobile-services-windows-phone-get-started" title="Windows Phone">Windows Phone</a>
	<a href="/zh-cn/documentation/articles/mobile-services-ios-get-started" title="iOS">iOS</a>
	<a href="/zh-cn/documentation/articles/mobile-services-android-get-started" title="Android">Android</a>
	<a href="/zh-cn/documentation/articles/mobile-services-html-get-started" title="HTML">HTML</a>
	<a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started" title="Xamarin.iOS">Xamarin.iOS</a>
	<a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started" title="Xamarin.Android">Xamarin.Android</a>
	<a href="/zh-cn/documentation/articles/partner-sencha-mobile-services-get-started/" title="Sencha">Sencha</a>
	<a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-phonegap-get-started/" title="PhoneGap">PhoneGap</a>
</div>

<div class="dev-center-tutorial-subselector">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started/" title=".NET backend">.NET 后端</a> | 
	<a href="/zh-cn/documentation/articles/mobile-services-windows-store-get-started/"  title="JavaScript backend" class="current">JavaScript 后端</a>
</div>

<div class="dev-onpage-video-clear clearfix">
<div class="dev-onpage-left-content">
<p>本教程说明如何使用 Azure 移动服务向 Windows 应用商店应用程序添加基于云的后端服务。</p>

<p>如果你更愿意观看视频，右侧的视频片段提供了与本教程相同的步骤。在视频中，Scott Guthrie 简要介绍了移动服务，并指导你完成创建第一个移动服务并从 Windows 应用商店应用程序连接到该服务的过程。</p>
</div>

<div class="dev-onpage-video-wrapper"><a href="http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Introduction-to-Windows-Azure-Mobile-Services" target="_blank" class="label">观看教程</a> <a style="background-image: url('/media/devcenter/mobile/videos/get-started-180x120.png') !important;" href="http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Introduction-to-Windows-Azure-Mobile-Services" target="_blank" class="dev-onpage-video"><span class="icon">播放视频</span></a> <span class="time">10:08</span></div>
</div>

在本教程中，你将要创建一个新的移动服务，以及一个在新移动服务中存储应用程序数据的简单*待办事项列表*应用程序。要创建的移动服务将为服务器端业务逻辑使用 JavaScript。若要创建允许你使用 Visual Studio 以受支持 .NET 语言编写服务器端业务逻辑的移动服务，请参阅本主题中的 .NET 后端版本。

以下是完成的应用程序的屏幕快照：

![][]

只有在完成本教程后，才可以学习有关 Windows 应用商店应用程序的所有其他移动服务教程。

> [WACOM.NOTE] 若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用][]。

> 本教程需要 Visual Studio 2013。若要使用 Visual Studio 2012 连接 Windows 应用商店应用程序，请按照[使用 Visual Studio 2012 的移动服务中的数据处理入门][]主题中的步骤进行操作。

## 创建新的移动服务

[WACOM.INCLUDE [mobile-services-create-new-service][]]

## 创建新的 Windows 应用商店应用程序

创建移动服务后，你可以在管理门户中遵照一个简易的快速入门项目来创建新应用程序或修改现有应用程序，以连接到你的移动服务。

在本部分中，你将要创建一个连接到移动服务的新的 Windows 应用商店应用程序。

1.  在管理门户中单击“移动服务”，然后单击你刚刚创建的移动服务 。

2.  在快速入门选项卡中，单击“选择平台”下的“Windows”，然后展开“创建新的 Windows 应用商店应用程序” 。

    ![][1]

    此时将显示三个简单步骤，描述如何创建与移动服务连接的 Windows 应用商店应用程序。

    ![][2]

3.  在本地计算机或虚拟机上下载并安装 [Visual Studio 2013 Express for Windows][]（如果尚未这么做）。

4.  单击“创建 TodoItem 表” 以创建用于存储应用程序数据的表。

5.  在“下载并运行应用程序” 下面，选择应用程序的语言，然后单击“下载” 。

    随即将会下载已连接到移动服务的示例*待办事项列表*应用程序的项目。将压缩的项目文件保存到本地计算机，并记下保存位置。

## 运行 Windows 应用程序

本教程的最后一个阶段是生成和运行你的新应用程序。

1.  浏览到你保存压缩项目文件的位置，在计算机上展开文件，并在 Visual Studio 2013 Express for Windows 中打开解决方案文件。

2.  按 "F5" 键重新生成项目并启动应用程序。

3.  在应用程序中的“插入 TodoItem”中键入有意义的文本（例如 *Complete the tutorial*），然后 单击“保存” 。

    ![][3]

    这样可向在 Azure 中托管的新移动服务发送 POST 请求。来自请求的数据被插入到 TodoItem 表。移动服务返回存储在表中的项，数据显示在应用程序的第二列中。

    > [WACOM.NOTE] 你可以查看访问你的移动服务以查询和插入数据的代码，这些代码在 MainPage.xaml.cs 文件（C\#/XAML 项目）或 default.js（JavaScript/HTML 项目）文件中。

4.  返回管理门户，单击“数据” 选项卡，然后单击“TodoItems” 表。

    ![][4]

    此时，你可以浏览应用程序在表中插入的数据。

    ![][5]

## 后续步骤

完成快速入门后，请了解如何在移动服务中执行其他重要任务：

-   "数据处理入门" ([C#][]/[JavaScript][])
    了解有关使用移动服务存储和查询数据的详细信息。

-   "身份验证入门" ([C#][6]/[JavaScript][7])
    了解如何使用标识提供者对应用程序用户进行身份验证。

-   "推送通知入门" ([C#][8]/[JavaScript][9])
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
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started/ ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-windows-store-get-started/ "JavaScript 后端"
  [观看教程]: http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Introduction-to-Windows-Azure-Mobile-Services
  []: ./media/mobile-services-windows-store-get-started/mobile-quickstart-completed.png
  [Azure 免费试用]: http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=A0E0E5C02&returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started%2F
  [使用 Visual Studio 2012 的移动服务中的数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-dotnet-vs2012/
  [mobile-services-create-new-service]: ../includes/mobile-services-create-new-service.md
  [1]: ./media/mobile-services-windows-store-get-started/mobile-portal-quickstart.png
  [2]: ./media/mobile-services-windows-store-get-started/mobile-quickstart-steps.png
  [Visual Studio 2013 Express for Windows]: http://go.microsoft.com/fwlink/?LinkId=257546
  [3]: ./media/mobile-services-windows-store-get-started/mobile-quickstart-startup.png
  [4]: ./media/mobile-services-windows-store-get-started/mobile-data-tab.png
  [5]: ./media/mobile-services-windows-store-get-started/mobile-data-browse.png
  [C#]: /zh-cn/develop/mobile/tutorials/get-started-with-data-dotnet
  [JavaScript]: /zh-cn/develop/mobile/tutorials/get-started-with-data-js
  [6]: /zh-cn/develop/mobile/tutorials/get-started-with-users-dotnet
  [7]: /zh-cn/develop/mobile/tutorials/get-started-with-users-js
  [8]: /zh-cn/develop/mobile/tutorials/get-started-with-push-dotnet
  [9]: /zh-cn/develop/mobile/tutorials/get-started-with-push-js
