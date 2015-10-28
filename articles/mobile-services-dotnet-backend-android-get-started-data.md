<properties 
	pageTitle="数据处理入门 (Android) | 移动开发人员中心" 
	description="了解如何开始使用移动服务来利用 Android 应用程序中的数据。" 
	services="mobile-services" 
	documentationCenter="android" 
	authors="RickSaling" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="07/29/2015" 
	wacn.date="10/03/2015"/>

# 将移动服务添加到现有应用程序

[AZURE.INCLUDE [mobile-services-selector-get-started-data](../includes/mobile-services-selector-get-started-data.md)]

本主题说明如何使用 Azure 移动服务作为 Android 应用程序的后端数据源。在本教程中，你将要创建一个新移动服务，为某个应用程序（该应用程序在内存中存储数据）下载一个 Eclipse Android 项目，将该移动服务与该应用程序相集成，并查看运行该应用程序时对数据所做的更改。

在本教程中创建的移动服务支持移动服务中的 .NET 运行时。这样，你便可以将 .NET 语言和 Visual Studio 用于移动服务中的服务器端业务逻辑。若要创建允许以 JavaScript 编写服务器端业务逻辑的移动服务，请参阅本主题的 [JavaScript 后端版本](/documentation/articles/mobile-services-android-get-started-data)。

> [AZURE.NOTE]如果你要查看本教程的 Eclipse 版本，请转到：[数据处理入门 (Eclipse)]。

若要完成本教程，您需要以下各项：

+ <a href="https://go.microsoft.com/fwLink/p/?LinkID=391934" target="_blank">Visual Studio 2013</a> Update 3 或更高版本。 

+ 一个 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial/)。

## <a name="create-service"></a>创建新的移动服务

[AZURE.INCLUDE [mobile-services-dotnet-backend-create-new-service](../includes/mobile-services-dotnet-backend-create-new-service.md)]

## <a name="download-the-service"></a>将服务下载到本地计算机

[AZURE.INCLUDE [mobile-services-download-service-locally](../includes/mobile-services-download-service-locally.md)]

## <a name="test-the-service"></a>测试移动服务

[AZURE.INCLUDE [mobile-services-dotnet-backend-test-local-service](../includes/mobile-services-dotnet-backend-test-local-service.md)]

## <a name="publish-the-service"></a>将移动服务发布到 Azure

[AZURE.INCLUDE [mobile-services-dotnet-backend-publish-service](../includes/mobile-services-dotnet-backend-publish-service.md)]

## <a name="download-app"></a>下载 GetStartedWithData 项目

### 获取示例代码

[AZURE.INCLUDE [mobile-services-dotnet-backend-create-new-service](../includes/download-android-sample-code.md)]

### 验证 Android SDK 版本

[AZURE.INCLUDE [mobile-services-verify-android-sdk-version](../includes/mobile-services-verify-android-sdk-version.md)]


### 检查并运行示例代码

[AZURE.INCLUDE [mobile-services-android-run-sample-code](../includes/mobile-services-android-run-sample-code.md)]

## <a name="update-app"></a>更新应用程序以使用移动服务进行数据访问

[AZURE.INCLUDE [mobile-services-android-getting-started-with-data](../includes/mobile-services-android-getting-started-with-data.md)]

## <a name="test-app"></a>针对发布的移动服务测试应用程序

更新应用程序以使用移动服务作为后端存储后，便可以使用 Android 模拟器或 Android 手机针对移动服务测试该应用程序。

1. 在“运行”菜单中，单击“运行应用程序”以启动项目。

	这将执行使用 Android SDK 构建的应用，该应用使用客户端库发送一个查询，该查询从你的移动服务返回项目。

2. 和前面一样，输入有意义的文本，然后单击“添加”。

   	此时会将一个新项作为 insert 发送到移动服务。

   您可以重新启动此应用，以查看更改是否已持久保存在 Azure 中的数据库内。你也可以使用 Azure 管理门户来检查数据库：后续两个步骤将会执行此操作以查看数据库中的更改。

3. 在 Azure 管理门户中，单击与移动服务关联的数据库对应的“管理”。

    ![](./media/mobile-services-dotnet-backend-android-get-started-data/manage-sql-azure-database.png)

4. 在管理门户中，执行查询以查看 Windows Store 应用所做的更改。你的查询应类似于以下查询，不过，请使用你的数据库名称而不是 `todolist`。

        SELECT * FROM [todolist].[todoitems]

    ![](./media/mobile-services-dotnet-backend-android-get-started-data/sql-azure-query.png)

针对 Android 的**数据处理入门**教程到此结束。


## <a name="next-steps"></a>后续步骤

本教程演示了有关如何使 Android 应用程序处理移动服务中的数据的基础知识。

接下来，请试着学习下列教程之一：

* [身份验证入门]<br/>了解如何对应用程序用户进行身份验证。

* [推送通知入门]<br/>了解如何向应用程序发送一条很基本的推送通知。

* [移动服务 Android 操作方法概念性参考](/documentation/articles/mobile-services-android-how-to-use-client-library)<br/>了解有关如何将移动服务与 Android 一起使用的详细信息。
  
<!-- Anchors. -->

[Create a new mobile service]: #create-service
[Download the service locally]: #download-the-service-locally
[Test the mobile service]: #test-the-service
[Download the GetStartedWithData project]: #download-app
[Update the app to use the mobile service for data access]: #update-app
[Test the Android App against the service hosted locally]: #test-locally-hosted
[Publish the mobile service to Azure]: #publish-mobile-service
[Test the Android App against the service hosted in Azure]: #test-azure-hosted
[Test the app against the published mobile service]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->

<!-- URLs. -->
[数据处理入门 (Eclipse)]: /documentation/articles/mobile-services-dotnet-backend-android-get-started-data-EC
[Get started with Mobile Services]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started
[身份验证入门]: /documentation/articles/mobile-services-dotnet-backend-android-get-started-users
[推送通知入门]: /documentation/articles/mobile-services-dotnet-backend-android-get-started-push
[Azure Management Portal]: https://manage.windowsazure.cn/
[Management Portal]: https://manage.windowsazure.cn/
[Mobile Services SDK]: http://go.microsoft.com/fwlink/p/?LinkId=257545
[Developer Code Samples site]: http://go.microsoft.com/fwlink/p/?LinkId=328660
  

<!---HONumber=71-->