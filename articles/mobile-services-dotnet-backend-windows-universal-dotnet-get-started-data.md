<properties
	pageTitle="将移动服务添加到现有的通用 Windows 应用商店应用 | Microsoft Azure"
	description="了解如何开始使用移动服务来利用 Windows 应用商店应用程序中的数据。"
	services="mobile-services"
	documentationCenter="windows"
	authors="ggailey777"
	manager="dwrede"
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="03/07/2016"
	wacn.date="04/18/2016"/>

# 将移动服务添加到现有应用程序

[AZURE.INCLUDE [mobile-services-selector-get-started-data](../includes/mobile-services-selector-get-started-data.md)]
 

##概述

本主题说明如何使用 Azure 移动服务作为 Windows 应用商店应用程序的后端数据源。在本教程中，你将要为某个应用程序（该应用程序在内存中存储数据）下载一个 Visual Studio 2013 项目，创建一个新的移动服务，将该移动服务与该应用程序相集成，并查看运行该应用程序时对数据所做的更改。

在本教程中创建的移动服务是一个 .NET 后端移动服务。借助 .NET 后端，你可以对移动服务中的服务器端业务逻辑使用 .NET 语言和 Visual Studio，并且可以在本地计算机上运行和调试移动服务。若要创建允许以 JavaScript 编写服务器端业务逻辑的移动服务，请参阅本主题中的 JavaScript 后端版本。

>[AZURE.NOTE]本主题说明如何使用 Visual Studio Professional 2013 Update 3 中的工具将新的移动服务连接到通用 Windows 应用程序。你可以使用相同的步骤将移动服务连接到 Windows 应用商店或 Windows Phone 应用商店 8.1 应用程序。若要将移动服务连接到 Windows Phone 8.0 或 Windows Phone Silverlight 8.1 应用程序，请参阅[针对 Windows Phone 的数据处理入门](/documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-data/)。

##先决条件

若要完成本教程，您需要以下各项：

* 有效的 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial/)。
* <a href="https://go.microsoft.com/fwLink/p/?LinkID=391934" target="_blank">Visual Studio 2013</a> Update 3 或更高版本。 

##下载 GetStartedWithData 项目

[AZURE.INCLUDE [mobile-services-windows-universal-dotnet-download-project](../includes/mobile-services-windows-universal-dotnet-download-project.md)]

##在 Visual Studio 中创建新的移动服务

[AZURE.INCLUDE [mobile-services-dotnet-backend-create-new-service-vs2013](../includes/mobile-services-dotnet-backend-create-new-service-vs2013.md)]

&nbsp;&nbsp;7.在解决方案资源管理器中，打开 GetStartedWithData.Shared 项目文件夹中的 App.xaml.cs 代码文件，并查看已添加到 Windows 应用商店应用程序条件编译块中 **App** 类的新静态字段，如以下示例中所示：

	public static Microsoft.WindowsAzure.MobileServices.MobileServiceClient 
	    todolistClient = new Microsoft.WindowsAzure.MobileServices.MobileServiceClient(
	        "https://todolist.azure-mobile.net/",
	        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX");
		

&nbsp;&nbsp;此代码通过使用 [MobileServiceClient](http://go.microsoft.com/fwlink/p/?LinkId=302030) 类的一个实例提供对应用中新移动服务的访问权限。客户端是通过提供新移动服务的 URI 和应用程序密钥来创建的。此静态字段可用于你的应用程序中的所有页面。

&nbsp;&nbsp;8.右键单击 Windows Phone 应用程序项目，单击“添加”，单击“连接的服务...”，选择刚刚创建的移动服务，然后单击“确定”。将在共享的 App.xaml.cs 文件中添加相同的代码，但这一次会在 Windows Phone 应用程序条件编译块中添加。

此时，Windows 应用商店应用程序和 Windows Phone 应用商店应用程序都已连接到新的移动服务。下一步是测试新的移动服务项目。


##在本地测试移动服务项目

[AZURE.INCLUDE [mobile-services-dotnet-backend-test-local-service-api-documentation](../includes/mobile-services-dotnet-backend-test-local-service-api-documentation.md)]


##更新应用程序以使用移动服务

在本部分中，你将要更新通用 Windows 应用程序，以将移动服务用作应用程序的后端服务。只需对 GetStartedWithData.Shared 项目文件夹中的 MainPage.cs 项目文件进行更改。

[AZURE.INCLUDE [mobile-services-windows-dotnet-update-data-app](../includes/mobile-services-windows-dotnet-update-data-app.md)]


##将移动服务发布到 Azure

[AZURE.INCLUDE [mobile-services-dotnet-backend-publish-service](../includes/mobile-services-dotnet-backend-publish-service.md)]


##测试在 Azure 中托管的移动服务

现在，我们可以针对 Azure 中托管的移动服务，测试这两个版本的通用 Windows 应用程序。

[AZURE.INCLUDE [mobile-services-windows-universal-test-app](../includes/mobile-services-windows-universal-test-app.md)]

##查看 SQL 数据库中存储的数据

[AZURE.INCLUDE [mobile-services-dotnet-backend-view-sql-data](../includes/mobile-services-dotnet-backend-view-sql-data.md)]
 
本教程到此结束。

##后续步骤

本教程演示了使一个通用的 Windows 应用程序项目，以使用移动服务中的数据的基础知识。建议你接下来阅读下列其他主题之一：

* [身份验证入门]
  <br/>了解如何对应用程序用户进行身份验证。

* [推送通知入门]
  <br/>了解如何向应用程序发送一条很基本的推送通知。

* [移动服务 C# 操作方法概念性参考 ](/documentation/articles/mobile-services-dotnet-how-to-use-client-library/)
  <br/>了解有关如何将移动服务与 .NET 一起使用的详细信息。
  

<!-- Images. -->



<!-- URLs. -->

[Validate and modify data with scripts]: /documentation/articles/mobile-services-windows-store-dotnet-validate-modify-data-server-scripts/
[Refine queries with paging]: /documentation/articles/mobile-services-windows-store-dotnet-add-paging-data/
[Get started with Mobile Services]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started/
[身份验证入门]: /documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-users/
[推送通知入门]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-push/
[Get started with offline data sync]: /documentation/articles/mobile-services-windows-store-dotnet-get-started-offline-data/

[Mobile Services SDK]: http://go.microsoft.com/fwlink/p/?LinkId=257545
[Developer Code Samples site]: http://go.microsoft.com/fwlink/p/?LinkID=510826
[Mobile Services .NET How-to Conceptual Reference]: /documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library/
[MobileServiceClient class]: http://go.microsoft.com/fwlink/p/?LinkId=302030
 

<!---HONumber=Mooncake_0118_2016-->