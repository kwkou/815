<properties
	pageTitle="适用于 Windows 应用商店 JavaScript 应用的移动服务入门 | Azure 移动服务"
	description="按照本教程开始使用 Azure 移动服务通过 JavaScript 进行 Windows 应用商店开发。"
	services="mobile-services"
	documentationCenter="windows"
	authors="ggailey777"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="mobile-services"
	ms.date="09/16/2015"
	wacn.date="10/22/2015"/>

# 移动服务入门

[AZURE.INCLUDE [mobile-services-selector-get-started](../includes/mobile-services-selector-get-started.md)]

本教程说明如何使用 Azure 移动服务向 Windows 应用商店 JavaScript 应用添加基于云的后端服务。

在本教程中，你将要创建一个新的移动服务，以及一个在新移动服务中存储应用程序数据的简单*待办事项列表*应用程序。要创建的移动服务将为服务器端业务逻辑使用 JavaScript。若要创建允许你使用 Visual Studio 以受支持 .NET 语言编写服务器端业务逻辑的移动服务，请参阅[本主题的 .NET 后端版本](/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started)。

[WACOM.INCLUDE [mobile-services-windows-universal-get-started](../includes/mobile-services-windows-universal-get-started.md)]

若要完成本教程，您需要以下各项：

* 有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial)。
* [Visual Studio 2013 Express for Windows] 

## 创建新的移动服务

[AZURE.INCLUDE [mobile-services-create-new-service](../includes/mobile-services-create-new-service.md)]

## 创建新的 Windows 应用商店应用程序

创建移动服务后，你可以在管理门户中按照简单的快速入门来创建一个新的可连接到移动服务的 Windows 应用商店 8.1 JavaScript 应用。

1.  在管理门户中单击“移动服务”，然后单击你刚刚创建的移动服务。

   
2. 在快速入门选项卡中，单击“选择平台”下的“Windows”，然后展开“创建新的 Windows 应用商店应用程序”。

   ![](./media/mobile-services-javascript-backend-windows-store-javascript-get-started/mobile-portal-quickstart.png)

3. 在本地计算机或虚拟机上下载并安装 [Visual Studio 2013][Visual Studio 2013 Express for Windows]（如果尚未这么做）。

4. 单击“创建 TodoItem 表”以创建用于存储应用程序数据的表。

5. 在“下载并运行应用”下，选择应用的语言，然后单击“下载”。

  	随即将会下载已连接到移动服务的示例*待办事项列表*应用程序的项目。将压缩的项目文件保存到本地计算机，并记下保存位置。

## 运行 Windows 应用程序

本教程的最后一个阶段是生成和运行你的新应用程序。

1. 浏览到您保存压缩项目文件的位置，在计算机上展开文件，并在 Visual Studio 中打开解决方案文件。

2. 按 **F5** 键以重新构建项目并启动此应用。

3. 在应用程序中的“插入 TodoItem”中键入有意义的文本（例如 *Complete the tutorial*），然后单击“保存”。

   	这样可向在 Azure 中托管的新移动服务发送 POST 请求。来自请求的数据被插入到 TodoItem 表。移动服务返回存储在表中的项，数据显示在应用的第二列中。

4. （可选）再次运行应用，并注意在应用启动后将从移动服务加载在上一步中保存的数据。
 
5. 返回管理门户，单击“数据”选项卡，然后单击 **TodoItem** 表。

   	这使您可以浏览此应用插入表中的数据。

>[AZURE.NOTE]你可以查看访问你的移动服务以查询和插入数据的代码，这些代码在 default.js 文件中。

<!-- Anchors. -->
[Getting started with Mobile Services]: #getting-started
[Create a new mobile service]: #create-new-service
[Define the mobile service instance]: #define-mobile-service-instance
[Next Steps]: #next-steps

<!-- Images. -->

<!-- URLs. -->
[Get started with data]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-universal-javascript-get-started-data
[Get started with authentication]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-users
[Get started with push notifications]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-get-started-push
[Visual Studio 2013 Express for Windows]: http://go.microsoft.com/fwlink/?LinkId=257546
[Mobile Services SDK]: http://go.microsoft.com/fwlink/?LinkId=257545
[Management Portal]: https://manage.windowsazure.cn/

<!---HONumber=74-->