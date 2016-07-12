<properties 
	pageTitle="适用于 Xamarin.Android 的移动服务入门 | Azure" 
	writer="craigd" 
	description="了解如何对 Xamarin.Android 应用程序使用 Azure 移动服务。" 
	documentationCenter="xamarin" 
	authors="lindydonna" 
	manager="dwrede" 
	editor="" 
	services="mobile-services"/>

<tags 
	ms.service="mobile-services" 
	ms.date="02/10/2016"
	wacn.date="05/23/2016"/>

#  <a name="getting-started"></a>移动服务入门

[AZURE.INCLUDE [mobile-services-selector-get-started](../includes/mobile-services-selector-get-started.md)]

本教程说明如何使用 Azure 移动服务向 Xamarin.Android 应用程序添加基于云的后端服务。在本教程中，你将要创建一个新的移动服务，以及一个在新移动服务中存储应用程序数据的简单待办事项列表应用程序。



以下是完成的应用程序的屏幕快照：

![][0]

完成本教程需要 XCode 和 Xamarin Studio（对于 OS X）或 Visual Studio（对于 Windows）以及联网的 Mac。[设置和安装 Visual Studio 和 Xamarin](https://msdn.microsoft.com/zh-cn/library/mt613162.aspx) 中提供了完整的安装说明。

下载的快速入门项目包含适用于 Xamarin.Android 的 Azure 移动服务组件。尽管此项目面向 Android 4.2 或更高版本，但移动服务 SDK 只需要 Android 2.2 或更高版本。

> [AZURE.IMPORTANT]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以注册 Azure 试用版并取得多达 10 个免费的移动服务，即使在试用期结束之后仍可继续使用这些服务。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial)。

##  <a name="create-new-service"></a>创建新的移动服务

[AZURE.INCLUDE [mobile-services-create-new-service](../includes/mobile-services-create-new-service.md)]

##  创建新的 Xamarin.Android 应用程序

创建移动服务后，你可以在 Azure 经典门户中遵照一个简易的快速入门项目来创建新应用程序或修改现有应用程序，以连接到你的移动服务。

在本部分中，你将要创建一个连接到移动服务的新的 Xamarin.Android 应用程序。

1.  在 [Azure 经典门户]中单击“移动服务”，然后单击你刚刚创建的移动服务。

2. 在快速启动选项卡中，单击“选择平台”下的“Xamarin.Android”，然后展开“创建新的 Android 应用程序”。

	![][6]

	此时将显示三个简单步骤，描述如何创建与移动服务连接的 Xamarin.Android 应用程序。

	![][7]

3. 单击“创建 TodoItem 表”以创建用于存储应用程序数据的表。

4. 在“下载并运行应用程序”下面单击“下载”。

	随即将会下载已连接到移动服务的示例_待办事项列表_应用程序的项目。将压缩的项目文件保存到本地计算机，并记下保存位置。

##  运行 Android 应用程序

本教程的最后一个阶段是生成和运行你的新应用程序。

1. 浏览到压缩的项目文件所保存到的位置，然后在计算机上展开这些文件。

2. 在 Xamarin Studio 或 Visual Studio 中，依次单击“文件”、“打开”，导航到解压缩的示例文件，然后选择“XamarinTodoQuickStart.Android.sln”以将其打开。

3. 按“运行”按钮生成项目并启动应用程序。系统将要求你选择模拟器或已连接的 USB 设备。

	> [AZURE.NOTE]若要在 Android 模拟器中运行项目，必须至少定义一个 Android 虚拟设备 (AVD)。使用 AVD 管理器创建和管理这些设备。

4. 在应用程序中键入有意义的文本（例如 _完成本教程_），然后单击“添加”。

	![][10]

	这样可向在 Azure 中托管的新移动服务发送 POST 请求。来自请求的数据被插入到 TodoItem 表。移动服务返回存储在表中的项，数据显示在列表中。

	> [AZURE.NOTE] 
   	你可以查看访问你的移动服务以查询和插入数据的代码，这些代码在 ToDoActivity.cs C# 文件中。

6. 返回 [Azure 经典门户]，单击“数据”选项卡，然后单击“TodoItems”表。

	![][11]

	这使您可以浏览此应用插入表中的数据。

	![][12]

##  <a name="next-steps"></a>后续步骤
完成快速入门后，请了解如何在移动服务中执行其他重要任务：

* [脱机数据同步入门]
了解如何快速开始使用脱机数据同步来使应用程序保持较高的响应能力和稳健性。

* [身份验证入门]
 了解如何使用标识提供程序对应用程序的用户进行身份验证。




[AZURE.INCLUDE [app-service-disqus-feedback-slug](../includes/app-service-disqus-feedback-slug.md)]

<!-- Anchors. -->
[Getting started with Mobile Services]: #getting-started
[Create a new mobile service]: #create-new-service
[Define the mobile service instance]: #define-mobile-service-instance
[Next Steps]: #next-steps

<!-- Images. -->
[0]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-quickstart-completed-android.png
[2]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-create.png
[3]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-create-page1.png
[4]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-create-page2.png
[5]: ./media/partner-xamarin-mobile-services-android-get-started/obile-services-selection.png
[6]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-portal-quickstart-xamarin-android.png
[7]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-quickstart-steps-xamarin-android.png
[8]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-xamarin-project-android-xs.png
[9]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-xamarin-project-android-vs.png
[10]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-quickstart-startup-android.png
[11]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-data-tab.png
[12]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-data-browse.png
[13]: ./media/partner-xamarin-mobile-services-android-get-started/mobile-services-diagram.png


<!-- URLs. -->
[Get started with data]: /documentation/articles/partner-xamarin-mobile-services-android-get-started-data/
[脱机数据同步入门]: /documentation/articles/mobile-services-xamarin-android-get-started-offline-data/
[身份验证入门]: /documentation/articles/partner-xamarin-mobile-services-android-get-started-users/

[Xamarin.Android]: http://xamarin.com/download
[Mobile Services Android SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[WindowsAzure.com]: http://www.windowsazure.com/
[Azure 经典门户]: https://manage.windowsazure.cn/

<!---HONumber=Mooncake_0118_2016-->