<properties pageTitle="Get Started with Azure Mobile Services for Android apps" metaKeywords="Azure android application, mobile service android, getting started Azure android, azure droid, getting started droid windows" description="Follow this tutorial to get started using Azure Mobile Services for Android development." metaCanonical="" services="" documentationCenter="Mobile" title="Get started with Mobile Services" authors="glenga" solutions="" manager="" editor="" />

<a name="getting-started"> </a>
# 移动服务入门

<div class="dev-center-tutorial-selector sublanding">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started" title="Windows Store C#">Windows 应用商店 C#</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started" title="Windows Phone">Windows Phone</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-get-started" title="iOS">iOS</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started" title="Android" class="current">Android</a>
</div>

<div class="dev-center-tutorial-subselector">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started/" title=".NET backend" class="current">.NET 后端</a> | 
	<a href="/zh-cn/documentation/articles/mobile-services-android-get-started/"  title="JavaScript backend">JavaScript 后端</a>
</div>

本教程说明如何使用 Azure 移动服务向 Android 应用程序添加基于云的后端服务。在本教程中，你将要创建一个新的移动服务，以及一个在新移动服务中存储应用程序数据的简单*待办事项列表*应用程序。要创建的移动服务将使用支持的 .NET 语言，你可以使用 Visual Studio 来提供服务器端业务逻辑和管理移动服务。若要创建允许以 JavaScript 编写服务器端业务逻辑的移动服务，请参阅本主题中的 [JavaScript 后端版本][]。

以下是完成的应用程序的屏幕快照：

![][0]

完成本教程需要你安装 [Android 开发人员工具][]，其中包含 Eclipse 集成开发环境 (IDE)、Android 开发人员工具 (ADT) 插件和最新的 Android 平台。需要使用 Android 4.2 或更高版本。

下载的快速入门项目包含适用于 Android 的移动服务 SDK。尽管此项目需要 Android 4.2 或更高版本，但移动服务 SDK 只需要 Android 2.2 或更高版本。

<div class="dev-callout"><b>说明</b>

<p>若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=AE564AB28" target="_blank">Azure 免费试用</a>。</p>
</div>

<a name="create-new-service"> </a>
## 创建新的移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-create-new-service](../includes/mobile-services-dotnet-backend-create-new-service.md)]

## 将移动服务下载到本地计算机

在创建移动服务后，请下载可在本地计算机或虚拟机上运行的个性化移动服务项目。

1.  单击刚刚创建的移动服务，在快速入门选项卡中单击“选择平台”下的“Android”，然后展开“创建新的 Android 应用程序” 。

    ![][1]

2.  如果你尚未安装 Visual Studio，请下载和安装 [Visual Studio Professional 2013][] 或更高版本。

3.  在“下载你的服务并将其发布到云”下面单击“下载” 。

    这样可以下载实现你的移动服务的 Visual Studio 项目。将压缩的项目文件保存到本地计算机上，并记下你保存它的位置。

4.  另外，请下载发布配置文件，将下载的文件保存到本地计算机，然后记下保存位置。

## 测试移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-test-local-service](../includes/mobile-services-dotnet-backend-test-local-service.md)]

## 发布移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-publish-service](../includes/mobile-services-dotnet-backend-publish-service.md)]

## 创建新的 Android 应用程序

在本部分中，你将要创建一个连接到移动服务的新的 Android 应用程序。

1.  在[管理门户][]中单击“移动服务”，然后单击你刚刚创建的移动服务 。

2.  在快速入门选项卡中，单击“选择平台”下的“Android”，然后展开“创建新的 Android 应用程序” 。

    ![][2]

3.  在本地计算机或虚拟机上下载并安装 [Android 开发人员工具][]（如果尚未这么做）。

4.  在“下载并运行应用程序”下面单击“下载” 。

    随即将会下载已连接到移动服务的示例*待办事项列表*应用程序的项目。将压缩的项目文件保存到本地计算机，并记下保存位置。

## 运行 Android 应用程序

本教程的最后一个阶段是生成和运行你的新应用程序。

1.  浏览到压缩的项目文件所保存到的位置，然后在计算机上展开这些文件。

2.  在 Eclipse 中，单击“文件” ，然后单击“导入” ，展开“Android” ，单击“现有 Android 代码到工作区” ，再单击“下一步”。 

    ![][3]

3.  单击“浏览”，浏览到已展开项目文件所在的位置，单击“确定”，并确认 TodoActivity 项目已选中，然后单击“完成” 。

    ![][4]

    这样可将项目文件导入当前工作区。

    ![][5]

4.  在“运行”菜单中，单击“运行” ，以便在 Android 模拟器中启动项目。

	<div class="dev-callout"><b>说明</b>

    <p>若要在 Android 模拟器中运行项目，必须至少定义一个 Android 虚拟设备 (AVD)。使用 AVD 管理器创建和管理这些设备。</p>
	</div>

5.  在应用程序中键入有意义的文本（例如 *Complete the tutorial*），然后单击“添加” 。

    ![][6]

    这样可向在 Azure 中托管的新移动服务发送 POST 请求。来自请求的数据被插入到 TodoItem 表。移动服务返回存储在表中的项，数据显示在列表中。

	<div class="dev-callout"><b>说明</b>

    <p>你可以查看访问你的移动服务以查询和插入数据的代码，这些代码在 ToDoActivity.java 文件中。</p>
	</div>

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-get-started "iOS"
  [Android]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started "Android"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started/ ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-android-get-started/ "JavaScript 后端"
  [JavaScript 后端版本]: /zh-cn/documentation/articles/mobile-services-android-get-started/
  [0]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-quickstart-completed-android.png
  [Android 开发人员工具]: https://go.microsoft.com/fwLink/p/?LinkID=280125
  [Azure 免费试用]: http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=AE564AB28
  [mobile-services-dotnet-backend-create-new-service]: ../includes/mobile-services-dotnet-backend-create-new-service.md
  [1]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-quickstart-steps-vs.png
  [Visual Studio Professional 2013]: https://go.microsoft.com/fwLink/p/?LinkID=391934
  [mobile-services-dotnet-backend-test-local-service]: ../includes/mobile-services-dotnet-backend-test-local-service.md
  [mobile-services-dotnet-backend-publish-service]: ../includes/mobile-services-dotnet-backend-publish-service.md
  [管理门户]: https://manage.windowsazure.cn/
  [2]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-quickstart-steps-android.png
  [3]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-services-import-android-workspace.png
  [4]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-services-import-android-project.png
  [5]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-eclipse-quickstart.png
  [6]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-quickstart-startup-android.png
