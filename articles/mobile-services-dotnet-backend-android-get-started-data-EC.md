<properties 
	pageTitle="数据处理入门 (Android) | 移动开发人员中心" 
	description="了解如何开始使用移动服务来利用 Android 应用程序中的数据。" 
	services="mobile-services" 
	documentationCenter="android" 
	authors="RickSaling" 
	manager="dwrede" 
	editor=""/>

<tags 
wacn.date="04/15/2015"

	ms.service="mobile-services" 
	ms.workload="mobile" 
	ms.tgt_pltfrm="mobile-android" 
	ms.devlang="java" 
	ms.topic="article" 
	ms.date="02/06/2015" 
	ms.author="ricksal"/>

# 将移动服务添加到现有应用程序

[AZURE.INCLUDE [mobile-services-selector-get-started-data](../includes/mobile-services-selector-get-started-data-EC.md)]

本主题说明如何使用 Azure 移动服务作为 Android 应用程序的后端数据源。在本教程中，你将要创建一个新移动服务，为某个应用程序（该应用程序在内存中存储数据）下载一个 Eclipse Android 项目，将该移动服务与该应用程序相集成，并查看运行该应用程序时对数据所做的更改。

在本教程中创建的移动服务支持移动服务中的 .NET 运行时。这样，你便可以将 .NET 语言和 Visual Studio 用于移动服务中的服务器端业务逻辑。若要创建允许以 JavaScript 编写服务器端业务逻辑的移动服务，请参阅本主题中的 [JavaScript 后端版本]。

> [AZURE.NOTE] 本教程需要安装 Visual Studio 2013。

本教程将指导你完成以下基本步骤：


1. [创建新的移动服务]
2. [在本地下载服务]
3. [测试移动服务]
4. [将移动服务发布到 Azure]
5. [下载 GetStartedWithData 项目]
4. [更新应用程序以使用移动服务进行数据访问]
5. [针对发布的移动服务测试应用程序]


> [AZURE.NOTE] 若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅[Azure 试用](/pricing/1rmb-trial/)。 


<h2><a name="create-service"></a>创建新的移动服务</h2>

[AZURE.INCLUDE [mobile-services-dotnet-backend-create-new-service](../includes/mobile-services-dotnet-backend-create-new-service.md)]


<h2><a name="download-the-service"></a>将服务下载到本地计算机</h2>

[AZURE.INCLUDE [mobile-services-download-service-locally](../includes/mobile-services-download-service-locally.md)]

<h2><a name="test-the-service"></a>测试移动服务</h2>

[AZURE.INCLUDE [mobile-services-dotnet-backend-test-local-service](../includes/mobile-services-dotnet-backend-test-local-service.md)]

<h2><a name="publish-the-service"></a>将移动服务发布到 Azure</h2>

[AZURE.INCLUDE [mobile-services-dotnet-backend-publish-service](../includes/mobile-services-dotnet-backend-publish-service.md)]

<h2><a name="download-app"></a>下载 GetStartedWithData 项目</h2>

### 获取示例代码

[AZURE.INCLUDE [mobile-services-dotnet-backend-create-new-service](../includes/download-android-sample-code-EC.md)]

### 验证 Android SDK 版本

[AZURE.INCLUDE [mobile-services-verify-android-sdk-version](../includes/mobile-services-verify-android-sdk-version-EC.md)]


### 检查并运行示例代码

[AZURE.INCLUDE [mobile-services-android-run-sample-code](../includes/mobile-services-android-run-sample-code-EC.md)]

<h2><a name="update-app"></a>更新应用程序以使用移动服务进行数据访问</h2>

[AZURE.INCLUDE [mobile-services-android-getting-started-with-data](../includes/mobile-services-android-getting-started-with-data-EC.md)]

<h2><a name="test-app"></a>针对发布的移动服务测试应用程序</h2>


更新应用程序以使用移动服务作为后端存储后，便可以使用 Android 模拟器或 Android 手机针对移动服务测试该应用程序。

1. 在"运行"菜单中，单击"运行"以启动项目。

	这将执行使用 Android SDK 构建的应用，该应用使用客户端库发送一个查询，该查询从你的移动服务返回项目。

5. 和前面一样，输入有意义的文本，然后单击"添加"。

   	此时会将一个新项作为 insert 发送到移动服务。

    你可以重新启动应用程序，以查看更改是否已持久保存在 Azure 中的数据库内。你也可以使用 Azure 管理门户来检查数据库：后续两个步骤将会执行此操作以查看数据库中的更改。


4. 在 Azure 管理门户中，单击与移动服务关联的数据库对应的"管理"。

    ![](./media/mobile-services-dotnet-backend-android-get-started-data/manage-sql-azure-database.png)

5. 在管理门户中，执行查询以查看 Windows Store 应用所做的更改。你的查询应类似于以下查询，不过，请使用你的数据库名称而不是 `todolist`。

        SELECT * FROM [todolist].[todoitems]

    ![](./media/mobile-services-dotnet-backend-android-get-started-data/sql-azure-query.png)

针对 Android 的**数据处理入门**教程到此结束。



## <a name="next-steps"> </a>后续步骤

本教程演示了有关如何使 Android 应用程序处理移动服务中的数据的基础知识。 

<!--Next, consider completing one of the following tutorials that is based on the GetStartedWithData app that you created in this tutorial:

* [Validate and modify data with scripts]
  <br/>Learn more about using server scripts in Mobile Services to validate and change data sent from your app.

* [Refine queries with paging]
  <br/>Learn how to use paging in queries to control the amount of data handled in a single request.

Once you have completed the data series, try
-->

请试着学习下列教程之一：

* [身份验证入门]
  <br/>了解如何对应用程序用户进行身份验证。

* [推送通知入门]
  <br/>了解如何向应用程序发送一条非常简单的推送通知。

* [移动服务 .NET 操作方法概念性参考]
  <br/>了解有关如何将移动服务与 .NET 一起使用的详细信息。
  
<!-- Anchors. -->

[创建新的移动服务]: #create-service
[在本地下载服务]: #download-the-service-locally
[测试移动服务]: #test-the-service
[下载 GetStartedWithData 项目]: #download-app
[更新应用程序以使用移动服务进行数据访问]: #update-app
[针对本地托管的服务测试 Android 应用程序]: #test-locally-hosted
[将移动服务发布到 Azure]: #publish-mobile-service
[针对 Azure 中托管的服务测试 Android 应用程序]: #test-azure-hosted
[针对发布的移动服务测试应用程序]: #test-app
[后续步骤]:#next-steps

<!-- Images. -->
[0]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/app-view.png
[1]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/mobile-data-sample-download-dotnet-vs13.png
[2]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/mobile-service-overview-page.png
[3]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/download-service-project.png
[4]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/add-service-project-to-solution.png
[5]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/download-publishing-profile.png
[6]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/add-existing-project-dialog.png
[7]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/vs-manage-nuget-packages.png
[8]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/manage-nuget-packages.png
[9]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/copy-mobileserviceclient-snippet.png
[10]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/vs-pasted-mobileserviceclient.png
[11]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/vs-build-solution.png
[12]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/vs-run-solution.png
[13]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/new-local-todoitem.png
[14]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/vs-show-local-table-data.png
[15]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/local-item-checked.png
[16]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/azure-items.png
[17]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/manage-sql-azure-database.png
[18]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/sql-azure-query.png

[20]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/vs-build-service-project.png
[21]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/vs-start-debug-service-project.png
[22]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/service-welcome-page.png
[23]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/iis-express-tray.png

[26]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/copy-service-and-packages-folder.png


<!-- URLs. -->
[使用脚本验证和修改数据]: /develop/mobile/tutorials/validate-modify-and-augment-data-dotnet
[使用分页优化查询]: /develop/mobile/tutorials/add-paging-to-data-dotnet
[移动服务入门]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started/
[身份验证入门]: /develop/mobile/tutorials/get-started-with-users-android
[推送通知入门]: /develop/mobile/tutorials/get-started-with-push-android
[JavaScript 和 HTML]: /develop/mobile/tutorials/get-started-with-data-js
[JavaScript 后端版本]: /develop/mobile/tutorials/get-started-with-data-android

[Azure 管理门户]: https://manage.windowsazure.cn/
[管理门户]: https://manage.windowsazure.cn/
[移动服务 SDK]: https://zumo.blob.core.windows.net/sdk/azuresdk-win8-v0.2.5.msi
[开发人员代码示例站点]:  http://code.msdn.microsoft.com/Connect-to-Windows-Azure-3a8fc1ee
[移动服务 .NET 操作方法概念性参考]: /develop/mobile/how-to-guides/work-with-net-client-library
[MobileServiceClient 类]: https://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.windowsazure.mobileservices.mobileserviceclient.aspx
[移动服务 .NET 操作方法概念性参考]: /documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library  

<!--HONumber=50-->