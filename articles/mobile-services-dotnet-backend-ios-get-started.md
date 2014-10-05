<properties pageTitle="Get Started with Azure Mobile Services for iOS apps" metaKeywords="Azure iOS application, mobile service iOS, getting started Azure iOS" description="Follow this tutorial to get started using Azure Mobile Services for iOS development. " metaCanonical="" services="" documentationCenter="Mobile" title="Get started with Mobile Services" authors="glenga" solutions="" manager="" editor="" />

<a name="getting-started"> </a>
# 移动服务入门

<div class="dev-center-tutorial-selector sublanding">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started" title="Windows Store C#">Windows 应用商店 C\#</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started" title="Windows Phone">Windows Phone</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-get-started" title="iOS" class="current">iOS</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started" title="Android">Android</a>
	<!--<a href="/zh-cn/documentation/articles/get-started-html" title="HTML">HTML</a>
	<a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started" title="Xamarin.iOS">Xamarin.iOS</a>
	<a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started" title="Xamarin.Android">Xamarin.Android</a>
	<a href="/zh-cn/documentation/articles/partner-sencha-mobile-services-get-started/" title="Sencha">Sencha</a>
	<a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-phonegap-get-started/" title="PhoneGap">PhoneGap</a>-->
</div>

<div class="dev-center-tutorial-subselector">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-get-started/" title=".NET backend" class="current">.NET 后端</a> | 
	<a href="/zh-cn/documentation/articles/mobile-services-ios-get-started/"  title="JavaScript backend" >JavaScript 后端</a>
</div>

本教程说明如何使用 Azure 移动服务向 iOS 应用程序添加基于云的后端服务。在本教程中，你将要创建一个新的移动服务，以及一个在新移动服务中存储应用程序数据的简单*待办事项列表*应用程序。要创建的移动服务将使用支持的 .NET 语言，你可以使用 Visual Studio 来提供服务器端业务逻辑和管理移动服务。若要创建允许以 JavaScript 编写服务器端业务逻辑的移动服务，请参阅本主题中的 [JavaScript 后端版本][]。

以下是完成的应用程序的屏幕快照：

![][]

完成本教程需要安装 XCode 4.5 和 iOS 5.0 或更高版本。

<div class="dev-callout"><b>说明</b>

<p>若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=AE564AB28&amp;returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started-ios%2F" target="_blank">Azure 免费试用</a>。</p>
</div>

<a name="create-new-service"> </a>
## 创建新的移动服务

[WACOM.INCLUDE [mobile-services-create-new-service][]]

## 将移动服务下载到本地计算机

在创建移动服务后，请下载可在本地计算机或虚拟机上运行的个性化移动服务项目。

1.  单击刚刚创建的移动服务，在快速入门选项卡中单击“选择平台”下的“iOS”，然后展开“创建新的 iOS 应用程序” 。

    ![][1]

2.  如果你尚未安装 Visual Studio，请下载和安装 Visual Studio Professional 2013 或更高版本。

3.  在“下载你的服务并将其发布到云”下面单击“下载” 。

    这样可以下载实现你的移动服务的 Visual Studio 项目。将压缩的项目文件保存到本地计算机上，并记下你保存它的位置。

4.  另外，请下载发布配置文件，将下载的文件保存到本地计算机，然后记下保存位置。

## 测试移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-test-local-service][]]

## 发布移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-publish-service][]]

## 创建新的 iOS 应用程序

在本部分中，你将要创建一个连接到移动服务的新的 iOS 应用程序。

1.  在管理门户中单击“移动服务”，然后单击你刚刚创建的移动服务 。

2.  在快速入门选项卡中，单击“选择平台”下的“iOS”，然后展开“创建新的 iOS 应用程序” 。

3.  下载并安装 [Xcode][] v4.4 或更高版本（如果尚未这么做）。

4.  在“下载并运行应用程序”下面单击“下载” 。

    随即将会下载已连接到移动服务的示例*待办事项列表*应用程序的项目，以及移动服务 iOS SDK。将压缩的项目文件保存到本地计算机上，并记下你保存它的位置。

## 运行新的 iOS 应用程序

[WACOM.INCLUDE [mobile-services-ios-run-app][]]

本主题说明了如何针对 Azure 中运行的移动服务运行新的客户端应用程序。在对本地计算机上运行的移动服务测试 iOS 应用程序之前，必须配置 Web 服务器和防火墙，以允许从 iOS 开发计算机进行访问。有关详细信息，请参阅[配置本地 Web 服务器以允许连接到本地移动服务][]。

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-get-started "iOS"
  [Android]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started "Android"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-get-started/ ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-ios-get-started/ "JavaScript 后端"
  [JavaScript 后端版本]: /zh-cn/documentation/articles/mobile-services-ios-get-started
  []: ./media/mobile-services-dotnet-backend-ios-get-started/mobile-quickstart-completed-ios.png
  [Azure 免费试用]: http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=AE564AB28&returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started-ios%2F
  [mobile-services-create-new-service]: ../includes/mobile-services-create-new-service.md
  [1]: ./media/mobile-services-dotnet-backend-ios-get-started/mobile-quickstart-steps-vs.png
  [mobile-services-dotnet-backend-test-local-service]: ../includes/mobile-services-dotnet-backend-test-local-service.md
  [mobile-services-dotnet-backend-publish-service]: ../includes/mobile-services-dotnet-backend-publish-service.md
  [Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
  [mobile-services-ios-run-app]: ../includes/mobile-services-ios-run-app.md
  [配置本地 Web 服务器以允许连接到本地移动服务]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-how-to-configure-iis-express
