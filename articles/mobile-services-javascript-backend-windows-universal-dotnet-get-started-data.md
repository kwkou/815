<properties 
	pageTitle="将移动服务添加到现有的通用 Windows 应用 | Windows Azure" 
	description="了解如何将现有的通用 Windows 应用连接到 Azure 移动服务。" 
	services="mobile-services" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="07/22/2015" 
	wacn.date="10/22/2015"/>

#  将移动服务添加到现有应用程序

[AZURE.INCLUDE [mobile-services-selector-get-started-data](../includes/mobile-services-selector-get-started-data.md)]

## 概述

此主题说明如何通过 Azure 移动服务来利用通用 Windows 应用程序中的数据。通用 Windows 应用程序解决方案包括 Windows 应用商店 8.1 和 Windows Phone 应用商店 8.1 应用程序的项目，以及常见的共享项目。有关详细信息，请参阅[生成面向 Windows 和 Windows Phone 的通用 Windows 应用程序](http://msdn.microsoft.com/zh-cn/library/windows/apps/xaml/dn609832.aspx)。

在本教程中，你将要下载一个可在内存中存储数据的 Visual Studio 2013 应用程序项目，创建一个新的移动服务，将该移动服务与该应用程序相集成，然后登录到 Azure 管理门户以查看运行该应用程序时对数据所做的更改。

>[AZURE.NOTE]本主题说明如何使用 Visual Studio Professional 2013 Update 3 中的工具将新的移动服务连接到通用 Windows 应用程序。你可以使用相同的步骤将移动服务连接到 Windows 应用商店或 Windows Phone 应用商店 8.1 应用程序。若要将移动服务连接到 Windows Phone 8.0 或 Windows Phone Silverlight 8.1 应用程序，请参阅[针对 Windows Phone 的数据处理入门](/documentation/articles/mobile-services-windows-phone-get-started-data)。

若要完成本教程，您需要以下各项：

* 有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial)。
* [Visual Studio Express 2013 for Windows](https://go.microsoft.com/fwLink/p/?LinkID=257546)（Update 2 或更高版本）。 

## <a name="download-app"></a>下载 GetStartedWithData 项目

[AZURE.INCLUDE [mobile-services-windows-universal-dotnet-download-project](../includes/mobile-services-windows-universal-dotnet-download-project.md)]
 

## <a name="create-service"></a>在 Visual Studio 中创建新的移动服务

[AZURE.INCLUDE [mobile-services-create-new-service-vs2013](../includes/mobile-services-create-new-service-vs2013.md)]

&nbsp;&nbsp;7.在解决方案资源管理器中，打开 GetStartedWithData.Shared 项目文件夹中的 App.xaml.cs 代码文件，并查看已添加到 Windows 应用商店应用程序条件编译块中 **App** 类的新静态字段，如以下示例中所示：

	public static Microsoft.WindowsAzure.MobileServices.MobileServiceClient 
	    todolistClient = new Microsoft.WindowsAzure.MobileServices.MobileServiceClient(
	        "https://todolist.azure-mobile.net/",
	        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX");

&nbsp;&nbsp;此代码通过使用 [MobileServiceClient] 类的一个实例提供对应用中新移动服务的访问权限。客户端是通过提供新移动服务的 URI 和应用程序密钥来创建的。此静态字段可用于你的应用程序中的所有页面。

&nbsp;&nbsp;8.右键单击 Windows Phone 应用程序项目，单击“添加”，单击“连接的服务...”，选择刚刚创建的移动服务，然后单击“确定”。将在共享的 App.xaml.cs 文件中添加相同的代码，但这一次会在 Windows Phone 应用程序条件编译块中添加。

此时，Windows 应用商店应用程序和 Windows Phone 应用商店应用程序都已连接到新的移动服务。下一步是在移动服务中创建一个新的 TodoItem 表。

## <a name="add-table"></a>将新表添加到移动服务

[AZURE.INCLUDE [mobile-services-create-new-table-vs2013](../includes/mobile-services-create-new-table-vs2013.md)]

## <a name="update-app"></a>更新应用程序以使用移动服务

[AZURE.INCLUDE [mobile-services-windows-dotnet-update-data-app](../includes/mobile-services-windows-dotnet-update-data-app.md)]

## <a name="test-azure-hosted"></a>测试在 Azure 中托管的移动服务

现在，我们可以针对 Azure 中托管的移动服务，测试这两个版本的通用 Windows 应用程序。

[AZURE.INCLUDE [mobile-services-windows-universal-test-app](../includes/mobile-services-windows-universal-test-app.md)]

&nbsp;&nbsp;4.在 [Azure 管理门户]中，单击“移动服务”，然后单击你的移动服务。

&nbsp;&nbsp;5.单击“数据”>“浏览”可以看到，**TodoItem** 表现在包含了数据以及移动服务生成的 ID 值，并且已在该表中自动添加了列，以匹配应用中的 TodoItem 类。
     
本教程到此结束。

##  <a name="next-steps"></a>后续步骤

本教程演示了有关如何使通用 Windows 应用程序处理移动服务中的数据的基础知识。建议你接下来阅读下列其他主题之一：

* [身份验证入门]<br/>
  了解如何对应用程序用户进行身份验证。

* [推送通知入门 ]<br/>
  了解如何向应用程序发送一条很基本的推送通知。

* [移动服务 C# 操作方法概念性参考 ](/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library)<br/>
  了解有关如何将移动服务与 .NET 一起使用的详细信息。
  
<!-- Anchors. -->

[Get the Windows Store app]: #download-app
[Create the mobile service from Visual Studio]: #create-service
[Add a data table for storage]: #add-table
[Update the app to use the mobile service]: #update-app
[Test the app against Mobile Services]: #test-app
[View uploaded data in the Azure Management Portal]: #view-data
[Next Steps]: #next-steps

<!-- Images. -->

<!-- URLs. -->
[Validate and modify data with scripts]: /documentation/articles/mobile-services-windows-store-dotnet-validate-modify-data-server-scripts
[Refine queries with paging]: /documentation/articles/mobile-services-windows-store-dotnet-add-paging-data
[Get started with Mobile Services]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started
[Get started with data]: /documentation/articles/mobile-services-windows-store-dotnet-get-started-data
[身份验证入门]: /documentation/articles/mobile-services-windows-store-dotnet-get-started-users
[推送通知入门 ]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push
[Mobile Services .NET How-to Conceptual Reference]: /documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library

[Azure 管理门户]: https://manage.windowsazure.cn/
[Management Portal]: https://manage.windowsazure.cn/
[Mobile Services SDK]: http://go.microsoft.com/fwlink/p/?LinkId=257545
[Developer Code Samples site]: http://go.microsoft.com/fwlink/p/?LinkID=510826

[MobileServiceClient]: http://go.microsoft.com/fwlink/p/?LinkId=302030
 

<!---HONumber=74-->