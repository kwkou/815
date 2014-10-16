<properties pageTitle="Get Started with Azure Mobile Services for HTML 5 apps" metaKeywords="" description="Follow this tutorial to get started using Azure Mobile Services for HTML development. " metaCanonical="" services="" documentationCenter="Mobile" title="Get started with Mobile Services" authors="glenga" solutions="" manager="" editor="" />

<a name="getting-started"> </a>
# 移动服务入门

<div class="dev-center-tutorial-selector sublanding">
	<a href="/zh-cn/documentation/articles/mobile-services-windows-store-get-started" title="Windows Store">Windows Store</a>
	<a href="/zh-cn/documentation/articles/mobile-services-windows-phone-get-started" title="Windows Phone">Windows Phone</a>
	<a href="/zh-cn/documentation/articles/mobile-services-ios-get-started" title="iOS">iOS</a>
	<a href="/zh-cn/documentation/articles/mobile-services-android-get-started" title="Android">Android</a>
	<a href="/zh-cn/documentation/articles/mobile-services-html-get-started" title="HTML" class="current">HTML</a>
	<a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started" title="Xamarin.iOS">Xamarin.iOS</a>
	<a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started" title="Xamarin.Android">Xamarin.Android</a>
	<a href="/zh-cn/documentation/articles/partner-sencha-mobile-services-get-started/" title="Sencha">Sencha</a>
	<a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-phonegap-get-started/" title="PhoneGap">PhoneGap</a>
</div>

<!--<div class="dev-center-tutorial-subselector">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-html-get-started/" title=".NET backend">.NET 后端</a> | 
	<a href="/zh-cn/documentation/articles/mobile-services-html-get-started/"  title="JavaScript backend" class="current">JavaScript 后端</a>
</div>-->

<div class="dev-onpage-video-clear clearfix">
<div class="dev-onpage-left-content">
<p>本教程说明如何使用 Azure 移动服务向 HTML 应用程序添加基于云的后端服务。在本教程中，你将要创建一个新的移动服务，以及一个在新移动服务中存储应用程序数据的简单*待办事项列表*应用程序。单击右侧的剪辑可观看本教程的视频版本。</p>
</div>

</div>
<div class="dev-onpage-video-wrapper"><a href="http://go.microsoft.com/fwlink/?LinkId=287040" target="_blank" class="label">观看教程</a> <a style="background-image: url('/media/devcenter/mobile/videos/mobile-html-get-started-180x120.png') !important;" href="http://go.microsoft.com/fwlink/?LinkId=287040" target="_blank" class="dev-onpage-video"><span class="icon">播放视频 </span></a> <span class="time">3:51</span></div>
</div>

以下是完成的应用程序的屏幕快照：

![][0]

只有在完成本教程后，才可以学习有关 HTML 应用程序的所有其他移动服务教程。

<div class="dev-callout"><b>说明</b>

<p>若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <div class="dev-callout"><strong>Note</strong> <p>To complete this tutorial, you need an Azure account. If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see <a href="http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started-html%2F" target="_blank">Azure 免费试用</a>。</p>
</div>

<a name="create-new-service"> </a>
### 其他要求

-   本教程要求你在本地计算机上运行下列 Web 服务器之一：

    -   "在 Windows 上"：IIS Express。可通过 [Microsoft Web 平台安装程序][]安装 IIS Express。
    -   "在 MacOS X 上"：Python。该服务器事先应已安装。
    -   "在 Linux 上"：Python。必须安装[最新版本的 Python][]。

    你可以使用任何 Web 服务器来托管应用程序，但是这些 Web 服务器必须受下载的脚本的支持。

-   支持 HTML5 的 Web 浏览器。

## 创建新的移动服务

[WACOM.INCLUDE [mobile-services-create-new-service](../includes/mobile-services-create-new-service.md)]

## 

## 创建新应用程序创建新的 HTML 应用程序

创建移动服务后，你可以在管理门户中遵照一个简易的快速入门项目来创建新应用程序或修改现有应用程序，以连接到你的移动服务。

在本部分中，你将要创建一个连接到移动服务的新 HTML 应用程序。

1.  在管理门户中单击“移动服务”，然后单击你刚刚创建的移动服务 。

2.  在快速入门选项卡中，单击“选择平台”下的“Windows”，然后展开“创建新的 HTML 应用程序” 。

    ![][1]

    此时将显示三个简单步骤，描述如何创建和托管与移动服务连接的 HTML 应用程序。

    ![][2]

3.  单击“创建 TodoItems 表”以创建用于存储应用程序数据的表 。

4.  在“下载并运行应用程序”下面单击“下载” 。

    随即将会下载已连接到移动服务的示例*待办事项列表*应用程序的网站文件。将压缩文件保存到本地计算机，并记下保存位置。

5.  在“配置”选项卡中，检查 `localhost` 是否已列在“跨域资源共享(CORS)”下的“允许来自主机名的请求”列表中 。如果未列出，请在“主机名”字段中键入 `localhost`，然后单击“保存” 。

    ![][3]

	<div class="dev-callout"><b>说明</b>

    <p>如果将快速入门应用程序部署到除 localhost 以外的 Web 服务器，则必须将该 Web 服务器的主机名添加到“允许来自主机名的请求”列表 。有关详细信息，请参阅<a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/dn155871.aspx" target="_blank">跨域资源共享</a>。</p>
	</div>

## 托管和运行 HTML 应用程序

本教程的最后一个阶段是在本地计算机上托管和运行你的新应用程序。

1.  浏览到压缩的项目文件所保存到的位置，在计算机上展开这些文件，然后启动 "server" 子文件夹中的下列命令文件之一。

    -   "launch-windows"（Windows 计算机）
    -   "launch-mac.command"（Mac OS X 计算机）
    -   "launch-linux.sh"（Linux 计算机）

	<div class="dev-callout"><b>说明</b>

    <p>在 Windows 计算机上，当 PowerShell 要求你确认是否要运行脚本时，请键入“R”。你的 Web 浏览器可能会警告你不要运行该脚本，因为它是从 Internet 下载的。如果出现此警告，你必须请求浏览器继续加载该脚本。</p>
	</div>

    随后将在本地计算机上启动用于托管新应用程序的 Web 服务器。

2.  在 Web 浏览器中打开 URL <http://localhost:8000/> 以启动该应用程序。

3.  在应用程序中的“输入新任务”中键入有意义的文本（例如 *Complete the tutorial*），然后单击“添加” 。

    ![][4]

    这样可向在 Azure 中托管的新移动服务发送 POST 请求。来自请求的数据被插入到 TodoItem 表。移动服务返回存储在表中的项，数据显示在应用程序的第二列中。

	<div class="dev-callout"><b>说明</b>

    <p>你可以查看访问你的移动服务以查询和插入数据的代码，这些代码在 app.js 文件中。</p>
	</div>

4.  返回管理门户，单击“数据” 选项卡，然后单击“TodoItems” 表。

    ![][5]

    此时，你可以浏览应用程序在表中插入的数据。

    ![][6]
<a name="next-steps"> </a>
## 后续步骤

完成快速入门后，请了解如何在移动服务中执行其他重要任务：

-   "[数据处理入门][]"
    了解有关使用移动服务存储和查询数据的详细信息。

-   "[从 HTML 应用程序调用自定义 API][]"
    将 HTML 应用程序连接到移动服务上托管的自定义 API。

-   "[身份验证入门][]"
    了解如何使用标识提供者对应用程序的用户进行身份验证。

-   "[移动服务 HTML/JavaScript 操作方法概念性参考][]"
    了解有关如何将移动服务与 HTML/JavaScript 一起使用的详细信息

  [Windows 应用商店]: /zh-cn/documentation/articles/mobile-services-windows-store-get-started "Windows 应用商店"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/mobile-services-ios-get-started "iOS"
  [Android]: /zh-cn/documentation/articles/mobile-services-android-get-started "Android"
  [HTML]: /zh-cn/documentation/articles/mobile-services-html-get-started "HTML"
  [Xamarin.iOS]: /zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started "Xamarin.Android"
  [Sencha]: /zh-cn/documentation/articles/partner-sencha-mobile-services-get-started/ "Sencha"
  [PhoneGap]: /zh-cn/documentation/articles/mobile-services-javascript-backend-phonegap-get-started/ "PhoneGap"
  [观看教程]: http://go.microsoft.com/fwlink/?LinkId=287040
  [0]: ./media/mobile-services-html-get-started/mobile-quickstart-completed-html.png
  [Azure 免费试用]: http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=A0E0E5C02&returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started-html%2F
  [Microsoft Web 平台安装程序]: http://go.microsoft.com/fwlink/p/?LinkId=286333
  [最新版本的 Python]: http://go.microsoft.com/fwlink/p/?LinkId=286342
  [mobile-services-create-new-service]: ../includes/mobile-services-create-new-service.md
  [1]: ./media/mobile-services-html-get-started/mobile-portal-quickstart-html.png
  [2]: ./media/mobile-services-html-get-started/mobile-quickstart-steps-html.png
  [3]: ./media/mobile-services-html-get-started/mobile-services-set-cors-localhost.png
  [跨域资源共享]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn155871.aspx
  [4]: ./media/mobile-services-html-get-started/mobile-quickstart-startup-html.png
  [5]: ./media/mobile-services-html-get-started/mobile-data-tab.png
  [6]: ./media/mobile-services-html-get-started/mobile-data-browse.png
  [数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-html
  [从 HTML 应用程序调用自定义 API]: /zh-cn/documentation/articles/mobile-services-html-call-custom-api
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-html
  [移动服务 HTML/JavaScript 操作方法概念性参考]: /zh-cn/develop/mobile/how-to-guides/work-with-html-js-client
