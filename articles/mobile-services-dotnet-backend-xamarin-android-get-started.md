<properties
	pageTitle="适用于 Xamarin.Android 应用的移动服务入门 | Azure"
	description="按照本教程进行操作，开始使用 Azure 移动服务进行 Xamarin Android 开发"
	services="mobile-services"
	documentationCenter="xamarin"
	authors="lindydonna"
	manager="dwrede"
	editor="mollybos"/>

<tags 
	ms.service="mobile-services" 
	ms.date="01/14/2016"
	wacn.date="05/23/2016"/>

# <a name="getting-started"></a>移动服务入门

[AZURE.INCLUDE [mobile-services-selector-get-started](../includes/mobile-services-selector-get-started.md)]

本教程说明如何使用 Azure 移动服务向 Xamarin Android 应用程序添加基于云的后端服务。在本教程中，你将要创建一个新的移动服务，以及一个在新移动服务中存储应用程序数据的简单_待办事项列表_应用程序。要创建的移动服务将使用支持的 .NET 语言，你可以使用 Visual Studio 来提供服务器端业务逻辑和管理移动服务。若要创建允许以 JavaScript 编写服务器端业务逻辑的移动服务，请参阅本主题的 [JavaScript 后端版本]。

>[AZURE.NOTE]本主题演示如何使用 Azure 经典门户创建新的移动服务项目。通过使用 Visual Studio 2013 Update 2，还可以向现有的 Visual Studio 解决方案添加新的移动服务项目。有关详细信息，请参阅[快速入门：添加移动服务（.NET 后端）](http://msdn.microsoft.com/zh-cn/library/windows/apps/dn629482.aspx)

以下是完成的应用程序的屏幕快照：

![][0]

只有在完成本教程后，才可以学习有关 Xamarin Android 应用程序的所有其他移动服务教程。

>[AZURE.NOTE]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以注册 Azure 试用版并取得多达 10 个免费的移动服务，即使在试用期结束之后仍可继续使用这些服务。有关详细信息，请参阅 <a href="/pricing/1rmb-trial">Azure 试用</a>。<br />本教程需要安装 <a href="https://go.microsoft.com/fwLink/p/?LinkID=257546" target="_blank">Visual Studio Professional 2013</a>。可以使用免费试用版。
>本教程需要安装 [Visual Studio Professional 2013](https://go.microsoft.com/fwLink/p/?LinkID=257546)。可以使用免费试用版。

## 创建新的移动服务

[AZURE.INCLUDE [mobile-services-dotnet-backend-create-new-service](../includes/mobile-services-dotnet-backend-create-new-service.md)]

## 创建新的 Xamarin Android 应用程序

创建移动服务后，你可以在经典门户中遵照一个简易的快速入门项目来创建新应用或修改现有应用，以连接到你的移动服务。

在本部分中，你将为移动服务下载新的 Xamarin.android 应用程序和服务项目。

1. 如果尚未进行此操作，请安装 Visual Studio with Xamarin。有关说明，可查阅[设置和安装 Visual Studio 和 Xamarin](https://msdn.microsoft.com/zh-cn/library/mt613162.aspx)。你还可以使用 Mac OS X 计算机上的 Xamarin Studio，请参阅 [Mac 用户的设置、安装和验证](https://msdn.microsoft.com/zh-cn/library/mt488770.aspx)。  
1. 在 [Azure 管理门户]中单击“移动服务”，然后单击你刚刚创建的移动服务。

2. 在快速入门选项卡中，单击“选择平台”下的“Xamarin”，然后展开“创建新的 Xamarin 应用程序”。

   	![][6]

   	此时将显示三个简单步骤，描述如何创建与移动服务连接的 Xamarin Android 应用程序。

  	![][7]

4. 在“下载你的服务并将其发布到云”下，选择“Android”并单击“下载”。

  	随即将会下载一个解决方案，其中包含移动服务的项目，以及已连接到移动服务的示例_待办事项列表_应用程序的项目。将压缩的项目文件保存到本地计算机，并记下保存位置。

6. 下载发布配置文件，将下载的文件保存到本地计算机，然后记下保存位置。

## 测试移动服务

[AZURE.INCLUDE [mobile-services-dotnet-backend-test-local-service](../includes/mobile-services-dotnet-backend-test-local-service.md)]

## 发布移动服务

[AZURE.INCLUDE [mobile-services-dotnet-backend-publish-service](../includes/mobile-services-dotnet-backend-publish-service.md)]

## 运行 Xamarin Android 应用程序

本教程的最后一个阶段是生成和运行你的新应用程序。

1. 在 Visual Studio 或 Xamarin Studio 中，导航到移动服务解决方案中的客户端项目。

2. 按“运行”按钮生成项目并启动应用程序。系统将要求你选择模拟器或已连接的 USB 设备。

	> [AZURE.NOTE]若要在 Android 模拟器中运行项目，必须至少定义一个 Android 虚拟设备 (AVD)。使用 AVD 管理器创建和管理这些设备。

3. 在应用中键入有意义的文本（例如 _完成本教程_），然后单击加号 (**+**) 图标。

	![][10]

	这样可向在 Azure 中托管的新移动服务发送 POST 请求。来自请求的数据被插入到 TodoItem 表。移动服务返回存储在表中的项，数据显示在列表中。

	> [AZURE.NOTE] 
   	你可以查看访问你的移动服务以查询和插入数据的代码，这些代码在 ToDoActivity.cs C# 文件中。
    
## 后续步骤
完成快速入门后，请了解如何在移动服务中执行其他重要任务：

* [脱机数据同步入门]
<br/>了解如何快速开始使用脱机数据同步来使应用程序保持较高的响应能力和稳健性。

* [身份验证入门]
<br/>了解如何使用标识提供程序对应用程序的用户进行身份验证。


* [移动服务 .NET 后端故障排除]
<br/>了解如何诊断和修复移动服务 .NET 后端可能会出现的问题。

[AZURE.INCLUDE [app-service-disqus-feedback-slug](../includes/app-service-disqus-feedback-slug.md)]

<!-- Anchors. -->
[Getting started with Mobile Services]: #getting-started
[Create a new mobile service]: #create-new-service
[Next Steps]: #next-steps



<!-- Images. -->
[0]: ./media/mobile-services-dotnet-backend-xamarin-android-get-started/mobile-quickstart-completed-android.png
[6]: ./media/mobile-services-dotnet-backend-xamarin-android-get-started/mobile-portal-quickstart-xamarin.png
[7]: ./media/mobile-services-dotnet-backend-xamarin-android-get-started/mobile-quickstart-steps-xamarin-android.png
[8]: ./media/mobile-services-dotnet-backend-xamarin-android-get-started/mobile-xamarin-project-android-vs.png
[9]: ./media/mobile-services-dotnet-backend-xamarin-android-get-started/mobile-xamarin-project-android-xs.png
[10]: ./media/mobile-services-dotnet-backend-xamarin-android-get-started/mobile-quickstart-startup-android.png

<!-- URLs. -->
[脱机数据同步入门]: /documentation/articles/mobile-services-xamarin-android-get-started-offline-data/
[身份验证入门]: /documentation/articles/mobile-services-dotnet-backend-xamarin-android-get-started-users/

[Visual Studio Professional 2013]: https://go.microsoft.com/fwLink/p/?LinkID=257546
[Mobile Services SDK]: http://go.microsoft.com/fwlink/?LinkId=257545
[JavaScript and HTML]: /documentation/articles/mobile-services-win8-javascript/
[Azure 管理门户]: https://manage.windowsazure.cn/
[JavaScript 后端版本]: /documentation/articles/mobile-services-android-get-started/
[移动服务 .NET 后端故障排除]: /documentation/articles/mobile-services-dotnet-backend-how-to-troubleshoot/

<!---HONumber=Mooncake_0516_2016-->