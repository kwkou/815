<properties linkid="develop-mobile-tutorials-get-started-wp8" urlDisplayName="Get Started (WP8)" pageTitle="Get Started with Azure Mobile Services for Windows Phone apps" metaKeywords="" description="Follow this tutorial to get started using Azure Mobile Services for Windows Phone development. " metaCanonical="" services="" documentationCenter="Mobile" title="Get started with Mobile Services" authors="glenga" solutions="" manager="" editor="" />

# <a name="getting-started"> </a>移动服务入门

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started" title="Windows Store C#">Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started" title="Windows Phone" class="current">Windows Phone</a><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-get-started" title="iOS">iOS</a>	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started" title="Android">Android</a>
	<!--<a href="/zh-cn/documentation/articles/get-started-android" title="Android">Android</a>
		<a href="/zh-cn/documentation/articles/get-started-html" title="HTML">HTML</a>
		<a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started" title="Xamarin.iOS">Xamarin.iOS</a>
		<a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started" title="Xamarin.Android">Xamarin.Android</a>
		<a href="/zh-cn/documentation/articles/partner-sencha-mobile-services-get-started/" title="Sencha">Sencha</a>
		<a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-phonegap-get-started/" title="PhoneGap">PhoneGap</a>-->
</div>

<div class="dev-center-tutorial-subselector">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started/" title=".NET backend" class="current">.NET 后端</a> | <a href="/zh-cn/documentation/articles/mobile-services-windows-phone-get-started/"  title="JavaScript backend" >JavaScript 后端</a>
</div>

本教程说明如何使用 Azure 移动服务向 Windows Phone 8 应用程序添加基于云的后端服务。在本教程中，你将要创建一个新的移动服务，以及一个在新移动服务中存储应用程序数据的简单*待办事项列表*应用程序。要创建的移动服务将使用支持的 .NET 语言，你可以使用 Visual Studio 来提供服务器端业务逻辑和管理移动服务。若要创建允许以 JavaScript 编写服务器端业务逻辑的移动服务，请参阅本主题中的 [JavaScript 后端版本][]。

以下是完成的应用程序的屏幕快照：

![][0]

> [WACOM.NOTE] 若要完成本教程，你需要一个启用了 Azure 移动服务功能的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用][]。
> 本教程需要安装 [Visual Studio Professional 2013][]。可以使用免费试用版。若要创建新的 Windows Phone 8.1 应用程序，必须安装 Visual Studio 2013 Update 2 或更高版本。

## <a name="create-new-service"> </a>创建新的移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-create-new-service](../includes/mobile-services-dotnet-backend-create-new-service.md)]

## 创建新的 Windows Phone 应用程序

创建移动服务后，你可以在管理门户中遵照一个简易的快速入门项目来创建新应用程序或修改现有应用程序，以连接到你的移动服务。

在本部分中，你将要创建一个连接到移动服务的新的 Windows Phone 8 应用程序。

1.  在管理门户中单击“移动服务”，然后单击你刚刚创建的移动服务 。

2.  在快速入门选项卡中，单击“选择平台”下的“Windows Phone 8”，然后展开“创建新的 Windows Phone 8 应用程序” 。

    ![][1]

    此时将显示三个简单步骤，描述如何创建与移动服务连接的 Windows Phone 应用程序。

    ![][2]

3.  在本地计算机或虚拟机上下载并安装 [Visual Studio Professional 2013][]（如果尚未这么做）。

4.  在“下载你的服务并将其发布到云”下面单击“下载” 。

    随即将会下载一个解决方案，其中包含移动服务的项目，以及已连接到移动服务的示例*待办事项列表*应用程序的项目。将压缩的项目文件保存到本地计算机，并记下保存位置。

5.  在“将服务发布到云”下面，下载发布配置文件，将下载的文件保存到本地计算机，然后记下保存位置 。

## 在本地计算机上测试移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-test-local-service](../includes/mobile-services-dotnet-backend-test-local-service.md)]

> [WACOM.NOTE] 需要完成其他一些配置步骤才能运行连接到本地服务的 Windows Phone 应用程序。我们不会在本主题中说明具体的操作，但你可以通过[如何从 Windows Phone 8 模拟器连接到本地 Web 服务][]来了解详细信息。

## 发布移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-publish-service](../includes/mobile-services-dotnet-backend-publish-service.md)]

<ol start="4">
<li><p>在 Windows Phone 应用程序项目中，打开 App.xaml.cs 文件，找到创建 [MobileServiceClient][] 实例的代码，注释掉使用 *localhost* 创建此客户端的代码，然后取消注释使用如下所示远程移动服务 URL 创建客户端的代码：</p>

        <pre><code>public static MobileServiceClient MobileService = new MobileServiceClient(
            "https://todolist.azure-mobile.net/",
            "XXXXXXX-APPLICATION-KEY-XXXXXXXX");</code></pre>

	<p>现在，客户端将会访问已发布到 Azure 的移动服务。</p></li>

<li><p>（可选）如果要创建 Windows Phone 8.1 应用程序，请在解决方案资源管理器中右键单击项目，单击“属性”，将“目标 Windows Phone 操作系统版本”设置为“Windows Phone 8.1”，然后单击“是”以升级项目 。</p></li>

<li><p>按 <b>F5</b> 键重新生成项目并启动应用程序。</p></li>

<li><p>在应用程序中键入有意义的文本（例如 *Complete the tutorial*），然后单击“保存” 。</p>

</p>这样可向在 Azure 中托管的新移动服务发送 POST 请求。来自请求的数据被插入到 TodoItem 表。移动服务返回存储在表中的项，数据显示在列表中。</p>
	<div class="dev-callout"> 
	<b>说明</b> 
   	<p>你可以查看访问你的移动服务以查询和插入数据的代码，这些代码在 MainPage.xaml.cs 文件中。</p> 
 	</div>
</li>
</ol>

![][3]

本主题说明了如何针对 Azure 中运行的移动服务运行新的客户端应用程序。在对本地计算机上运行的移动服务测试 Windows Phone 应用程序之前，必须配置 Web 服务器和防火墙，以允许从 Windows Phone 设备或模拟器进行访问。有关详细信息，请参阅[配置本地 Web 服务器以允许连接到本地移动服务][]。

## <a name="next-steps"> </a>后续步骤
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
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started/ ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started/ "JavaScript 后端"
  [JavaScript 后端版本]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started
  [0]: ./media/mobile-services-windows-phone-get-started/mobile-quickstart-completed-wp8.png
  [Azure 免费试用]: http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=A30A4DDE2&returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdocumentation%2Farticles%2Fmobile-services-dotnet-backend-windows-phone-get-started%2F
  [Visual Studio Professional 2013]: https://go.microsoft.com/fwLink/p/?LinkID=257546
  [mobile-services-dotnet-backend-create-new-service]: ../includes/mobile-services-dotnet-backend-create-new-service.md
  [1]: ./media/mobile-services-dotnet-backend-windows-phone-get-started/mobile-portal-quickstart-wp8.png
  [2]: ./media/mobile-services-dotnet-backend-windows-phone-get-started/mobile-quickstart-steps-wp8.png
  [mobile-services-dotnet-backend-test-local-service]: ../includes/mobile-services-dotnet-backend-test-local-service.md
  [如何从 Windows Phone 8 模拟器连接到本地 Web 服务]: http://go.microsoft.com/fwlink/p/?LinkId=391930
  [mobile-services-dotnet-backend-publish-service]: ../includes/mobile-services-dotnet-backend-publish-service.md
  [MobileServiceClient]: http://msdn.microsoft.com/zh-cn/library/Windowsazure/microsoft.windowsazure.mobileservices.mobileserviceclient.aspx
  [3]: ./media/mobile-services-dotnet-backend-windows-phone-get-started/mobile-quickstart-startup-wp8.png
  [配置本地 Web 服务器以允许连接到本地移动服务]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-how-to-configure-iis-express
  [数据处理入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-data
  [身份验证入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-users
  [推送通知入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-push
