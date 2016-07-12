<properties
	pageTitle="用于 Xamarin iOS 应用的移动服务入门 | Azure"
	description="按照本教程进行操作，开始使用 Azure 移动服务进行 Xamarin iOS 开发。"
	services="mobile-services"
	documentationCenter="xamarin"
	authors="conceptdev"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="mobile-services"
	ms.date="03/16/2016"
	wacn.date="05/23/2016"/>

# <a name="getting-started"></a>移动服务入门
[AZURE.INCLUDE [mobile-services-selector-get-started](../includes/mobile-services-selector-get-started.md)]

以下是完成的应用程序的屏幕快照：

![][0]

完成本教程需要 XCode 和 Xamarin Studio（对于 OS X）或 Visual Studio（对于 Windows）以及联网的 Mac。[设置和安装 Visual Studio 和 Xamarin](https://msdn.microsoft.com/zh-cn/library/mt613162.aspx) 中提供了完整的安装说明。

> [AZURE.IMPORTANT]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以注册 Azure 试用版并取得多达 10 个免费的移动服务，即使在试用期结束之后仍可继续使用这些服务。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial)。

##  <a name="create-new-service"></a>创建新的移动服务

[AZURE.INCLUDE [mobile-services-create-new-service](../includes/mobile-services-create-new-service.md)]

##  创建新的 Xamarin iOS 应用程序

创建移动服务后，你可以在 Azure 经典门户中遵照一个简易的快速入门项目来创建新应用程序或修改现有应用程序，以连接到你的移动服务。

在本部分中，你将要创建一个连接到移动服务的新的 Xamarin.iOS 应用程序。

1.  在 [Azure 经典门户]中单击“移动服务”，然后单击你刚刚创建的移动服务。

2. 在快速启动选项卡中，单击“选择平台”下的“Xamarin.iOS”，然后展开“创建新的 Xamarin.iOS 应用程序”。

	![][6]

	此时将显示三个简单步骤，描述如何创建与移动服务连接的 Xamarin.iOS 应用程序。

  	![][7]

3. 如果尚未这样做，现在请下载并安装 Xcode（我们建议安装最新版本 Xcode 6.0 或更高版本）和 [Xamarin Studio]。

4. 单击“创建 TodoItem 表”以创建用于存储应用程序数据的表。

5. 在“下载并运行应用程序”下面单击“下载”。

	随即将会下载示例_待办事项列表_应用程序的项目，该应用程序已连接到移动服务，并引用 Xamarin.iOS 的 Azure 移动服务组件。将压缩的项目文件保存到本地计算机，并记下保存位置。

##  运行新的 Xamarin.iOS 应用程序

本教程的最后一个阶段是生成和运行你的新应用程序。

1. 浏览到你保存压缩项目文件的位置，在计算机上展开文件，并使用 Xamarin Studio 或 Visual Studio 打开 **XamarinTodoQuickStart.iOS.sln** 解决方案文件。

	![][8]

	![][9]

2. 按“运行”按钮以生成项目，并在 iPhone 模拟器中启动应用，这是此项目的默认设置。

3. 在应用中键入有意义的文本（例如 _完成本教程_），然后单击加号 (**+**) 图标。

	![][10]

	这样可向在 Azure 中托管的新移动服务发送 POST 请求。来自请求的数据被插入到 TodoItem 表。移动服务返回存储在表中的项，数据显示在列表中。

	> [AZURE.NOTE]你可以查看访问你的移动服务以查询和插入数据的代码，这些代码在 TodoService.cs C# 文件中。

4. 返回 [Azure 经典门户]，单击“数据”选项卡，然后单击“TodoItems”表。

	![][11]

	这使您可以浏览此应用插入表中的数据。

	![][12]


##  后续步骤
完成快速入门后，请了解如何在移动服务中执行其他重要任务：

* [脱机数据同步入门]
了解如何快速开始使用脱机数据同步来使应用程序保持较高的响应能力和稳健性。

* [身份验证入门] 
了解如何使用标识提供程序对应用程序的用户进行身份验证。

* [推送通知入门] 
了解如何向应用程序发送一条很基本的推送通知。




[AZURE.INCLUDE [app-service-disqus-feedback-slug](../includes/app-service-disqus-feedback-slug.md)]

<!-- Anchors. -->
[Getting started with Mobile Services]: #getting-started
[Create a new mobile service]: #create-new-service
[Define the mobile service instance]: #define-mobile-service-instance
[Next Steps]: #next-steps

<!-- Images. -->
[0]: ./media/partner-xamarin-mobile-services-ios-get-started/mobile-quickstart-completed-ios.png
[6]: ./media/partner-xamarin-mobile-services-ios-get-started/mobile-portal-quickstart-xamarin-ios.png
[7]: ./media/partner-xamarin-mobile-services-ios-get-started/mobile-quickstart-steps-xamarin-ios.png
[8]: ./media/partner-xamarin-mobile-services-ios-get-started/mobile-xamarin-project-ios-xs.png
[9]: ./media/partner-xamarin-mobile-services-ios-get-started/mobile-xamarin-project-ios-vs.png
[10]: ./media/partner-xamarin-mobile-services-ios-get-started/mobile-quickstart-startup-ios.png
[11]: ./media/partner-xamarin-mobile-services-ios-get-started/mobile-data-tab.png
[12]: ./media/partner-xamarin-mobile-services-ios-get-started/mobile-data-browse.png


<!-- URLs. -->
[脱机数据同步入门]: /documentation/articles/mobile-services-xamarin-ios-get-started-offline-data/
[身份验证入门]: /documentation/articles/partner-xamarin-mobile-services-ios-get-started-users/
[推送通知入门]: /documentation/articles/partner-xamarin-mobile-services-ios-get-started-push/

[Xamarin Studio]: http://xamarin.com/download
[Mobile Services iOS SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533

[Azure 经典门户]: https://manage.windowsazure.cn/

<!---HONumber=Mooncake_0118_2016-->