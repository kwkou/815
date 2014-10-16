<properties linkid="develop-mobile-tutorials-get-started" pageTitle="Get Started with Mobile Services for Windows Store apps | Mobile Dev Center" metaKeywords="" description="Follow this tutorial to get started using Azure Mobile Services for Windows Store development in C#, VB, or JavaScript. " metaCanonical="" services="" documentationCenter="Mobile" title="Get started with Mobile Services" authors="glenga" solutions="" manager="" editor="" />

# <a name="getting-started"> </a>移动服务入门

<div class="dev-center-tutorial-selector sublanding">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started" title="Windows Store C#" class="current">Windows 应用商店 C#</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started" title="Windows Phone">Windows Phone</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-get-started" title="iOS">iOS</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started" title="Android">Android</a>
	<!--<a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started" title="Xamarin.iOS">Xamarin.iOS</a>
	<a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started" title="Xamarin.Android">Xamarin.Android</a>
	<a href="/zh-cn/documentation/articles/partner-sencha-mobile-services-get-started/" title="Sencha">Sencha</a>
	<a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-phonegap-get-started/" title="PhoneGap">PhoneGap</a>-->
</div>

<div class="dev-center-tutorial-subselector">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started/" title=".NET backend" class="current">.NET 后端</a> | <a href="/zh-cn/documentation/articles/mobile-services-windows-store-get-started/"  title="JavaScript backend" >JavaScript 后端</a>
</div>

本教程说明如何使用 Azure 移动服务向 Windows 应用商店应用程序添加基于云的后端服务。在本教程中，你将要创建一个新的移动服务，以及一个在新移动服务中存储应用程序数据的简单*待办事项列表*应用程序。要创建的移动服务将使用支持的 .NET 语言，你可以使用 Visual Studio 来提供服务器端业务逻辑和管理移动服务。若要创建允许以 JavaScript 编写服务器端业务逻辑的移动服务，请参阅本主题中的 [JavaScript 后端版本][]。

> [WACOM.NOTE] 本主题说明如何使用 Azure 管理门户创建新的移动服务项目和 Windows 应用商店应用程序。通过使用 Visual Studio 2013 Update 2，还可以向现有的 Visual Studio 解决方案添加新的移动服务项目。有关详细信息，请参阅[快速入门：添加移动服务（.NET 后端）][]

以下是完成的应用程序的屏幕快照：

![][0]

只有在完成本教程后，才可以学习有关 Windows 应用商店应用程序的所有其他移动服务教程。

> [WACOM.NOTE] 若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用][]。
> 本教程需要安装 [Visual Studio Professional 2013][]。可以使用免费试用版。

## 创建新的移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-create-new-service][]]

## 创建新的 Windows 应用商店应用程序

创建移动服务后，你可以在管理门户中遵照一个简易的快速入门项目来创建新应用程序或修改现有应用程序，以连接到你的移动服务。

在本部分中，你将要创建一个连接到移动服务的新的 Windows 应用商店应用程序。

1.  在管理门户中单击“移动服务”，然后单击你刚刚创建的移动服务 。

2.  在快速入门选项卡中，单击“选择平台”下的“Windows”，然后展开“创建新的 Windows 应用商店应用程序” 。

    ![][1]

    此时将显示三个简单步骤，描述如何创建与移动服务连接的 Windows 应用商店应用程序。

    ![][2]

3.  在本地计算机或虚拟机上下载并安装 [Visual Studio Professional 2013][]（如果尚未这么做）。

4.  在“下载并在本地运行应用程序和服务”下面，选择 Windows 应用商店应用程序的语言，然后单击“下载” 。

    随即将会下载一个解决方案，其中包含移动服务的项目，以及已连接到移动服务的示例*待办事项列表*应用程序的项目。将压缩的项目文件保存到本地计算机，并记下保存位置。

5.  在“将服务发布到云”下面，下载发布配置文件，将下载的文件保存到本地计算机，然后记下保存位置 。

## 针对本地移动服务测试应用程序

[WACOM.INCLUDE [mobile-services-dotnet-backend-test-local-service-dotnet](../includes/mobile-services-dotnet-backend-test-local-service-dotnet.md)]

> [WACOM.NOTE] 你可以查看访问你的移动服务以查询和插入数据的代码，这些代码在 MainPage.xaml.cs 文件中。

## 发布移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-publish-service](../includes/mobile-services-dotnet-backend-publish-service.md)]

<ol start="4">
<li><p>在 Windows 应用商店应用程序项目中，打开 App.xaml.cs 文件，找到创建 [MobileServiceClient][] 实例的代码，注释掉使用 *localhost* 创建此客户端的代码，然后取消注释使用如下所示远程移动服务 URL 创建客户端的代码：<p>

        <pre><code>public static MobileServiceClient MobileService = new MobileServiceClient(
            "https://todolist.azure-mobile.net/",
            "XXXX-APPLICATION-KEY-XXXXX");</code></pre>

    <p>现在，客户端将会访问已发布到 Azure 的移动服务。</p></li>

<li><p>按 <b>F5</b> 键重新生成项目并启动应用程序。</p></li>

<li><p>在应用程序中的“插入 TodoItem”中键入有意义的文本（例如 *Complete the tutorial*），然后 单击“保存” 。</p>

    <p>这样可向在 Azure 中托管的新移动服务发送 POST 请求。</p>
</li>
</ol>

![][3]

## 后续步骤

完成快速入门后，请了解如何在移动服务中执行其他重要任务：

-   [数据处理入门][]
    了解有关使用移动服务存储和查询数据的详细信息。

-   [身份验证入门][]
    了解如何使用标识提供者对应用程序的用户进行身份验证。

-   [推送通知入门][]
    了解如何向应用程序发送一条非常简单的推送通知。

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-get-started "iOS"
  [Android]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started "Android"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started/ ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-windows-store-get-started/ "JavaScript 后端"
  [JavaScript 后端版本]: /zh-cn/documentation/articles/mobile-services-windows-store-get-started
  [快速入门：添加移动服务（.NET 后端）]: http://msdn.microsoft.com/zh-cn/library/windows/apps/dn629482.aspx
  [0]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started/mobile-quickstart-completed.png
  [Azure 免费试用]: http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=A0E0E5C02&returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdocumentation%2Farticles%2Fmobile-services-dotnet-backend-windows-store-get-started%2F
  [Visual Studio Professional 2013]: https://go.microsoft.com/fwLink/p/?LinkID=257546
  [mobile-services-dotnet-backend-create-new-service]: ../includes/mobile-services-dotnet-backend-create-new-service.md
  [1]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started/mobile-portal-quickstart.png
  [2]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started/mobile-quickstart-steps.png
  [mobile-services-dotnet-backend-test-local-service-dotnet]: ../includes/mobile-services-dotnet-backend-test-local-service-dotnet.md
  [mobile-services-dotnet-backend-publish-service]: ../includes/mobile-services-dotnet-backend-publish-service.md
  [MobileServiceClient]: http://msdn.microsoft.com/zh-cn/library/Windowsazure/microsoft.windowsazure.mobileservices.mobileserviceclient.aspx
  [3]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started/mobile-quickstart-startup.png
  [数据处理入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data
  [身份验证入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-users
  [推送通知入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-push
