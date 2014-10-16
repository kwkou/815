<properties linkid="develop-mobile-tutorials-dotnet-backend-get-started-with-data-android" urlDisplayName="Get Started with Data" pageTitle="Get started with data (Android) | Mobile Dev Center" metaKeywords="" description="Learn how to get started using Mobile Services to leverage data in your Android app." metaCanonical="" services="" documentationCenter="Mobile" title="Get started with data in Mobile Services" authors="ricksal" solutions="" manager="dwrede" editor="" />

# 移动服务中的数据处理入门

<div class="dev-center-tutorial-selector sublanding">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/" title="Windows Store C#">Windows 应用商店 C#</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-data/" title="Windows Phone">Windows Phone</a>
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started-data/" title="Android" class="current">Android</a>

</div>

<div class="dev-center-tutorial-subselector">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started-data/" title=".NET backend" class="current">.NET 后端</a> | 
	<a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-android/"  title="JavaScript backend">JavaScript 后端</a>
</div>

本主题说明如何使用 Azure 移动服务作为 Android 应用程序的后端数据源。在本教程中，你将要创建一个新移动服务，为某个应用程序（该应用程序在内存中存储数据）下载一个 Eclipse Android 项目，将该移动服务与该应用程序相集成，并查看运行该应用程序时对数据所做的更改。

在本教程中创建的移动服务支持移动服务中的 .NET 运行时。这样，你便可以将 .NET 语言和 Visual Studio 用于移动服务中的服务器端业务逻辑。若要创建允许以 JavaScript 编写服务器端业务逻辑的移动服务，请参阅本主题中的 [JavaScript 后端版本][]。

<div class="dev-callout"><b>说明</b>

<p>本教程需要安装 Visual Studio 2013。</p>
</div>

本教程将指导你完成以下基本步骤：

1.  [创建新的移动服务][]
2.  [在本地下载服务][]
3.  [测试移动服务][]
4.  [将移动服务发布到 Azure][]
5.  [下载 GetStartedWithData 项目][]
6.  [更新应用程序以使用移动服务进行数据访问][]
7.  [针对发布的移动服务测试应用程序][]

<div class="dev-callout"><b>说明</b>

若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=AE564AB28&amp;returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fzh-cn%2Fdocumentation%2Farticles%2Fmobile-services-dotnet-backend-windows-store-dotnet-get-started-data%2F" target="_blank">Azure 免费试用</a>。
</div>

<a name="create-service"></a>
## 创建新的移动服务创建新的移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-create-new-service](../includes/mobile-services-dotnet-backend-create-new-service.md)]

<a name="download-the-service"></a>
## 下载服务将服务下载到本地计算机

[WACOM.INCLUDE [mobile-services-download-service-locally](../includes/mobile-services-download-service-locally.md)]

<a name="test-the-service"></a>
## 测试服务测试移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-test-local-service](../includes/mobile-services-dotnet-backend-test-local-service.md)]

<a name="publish-the-service"></a>
## 发布服务将移动服务发布到 Azure

[WACOM.INCLUDE [mobile-services-dotnet-backend-publish-service](../includes/mobile-services-dotnet-backend-publish-service.md)]

<a name="download-app"></a>
## 下载项目下载 GetStartedWithData 项目

### 获取示例代码

[WACOM.INCLUDE [mobile-services-dotnet-backend-create-new-service](../includes/download-android-sample-code.md)]

### 验证 Android SDK 版本

[WACOM.INCLUDE [mobile-services-verify-android-sdk-version](../includes/mobile-services-verify-android-sdk-version.md)]

### 检查并运行示例代码

[WACOM.INCLUDE [mobile-services-android-run-sample-code](../includes/mobile-services-android-run-sample-code.md)]

<a name="update-app"></a>
## 更新应用程序更新应用程序以使用移动服务进行数据访问

[WACOM.INCLUDE [mobile-services-android-getting-started-with-data](../includes/mobile-services-android-getting-started-with-data.md)]

<a name="test-app"></a>
## 测试应用程序针对发布的移动服务测试应用程序

更新应用程序以使用移动服务作为后端存储后，便可以使用 Android 模拟器或 Android 手机针对移动服务测试该应用程序。

1.  在“运行” 菜单中，单击“运行”以启动项目 。

    这将执行使用 Android SDK 构建的应用程序，该应用程序使用客户端库发送一个查询，该查询从你的移动服务返回项目。

2.  和前面一样，输入有意义的文本，然后单击“添加” 。

    此时会将一个新项作为 insert 发送到移动服务。

    你可以重新启动应用程序，以查看更改是否已持久保存在 Azure 中的数据库内。你也可以使用 Azure 管理门户检查数据库：后面的两个步骤将会执行此操作以查看数据库中的更改。

3.  在 Azure 管理门户中，单击与你的移动服务关联的数据库对应的“管理”。

    ![][0]

4.  在管理门户中，执行一个查询以查看 Windows 应用商店应用程序所做的更改。你的查询应类似于以下查询，不过，请使用你的数据库名称而不是 `todolist`。

        SELECT * FROM [todolist].[todoitems]

    ![][2]

针对 Android 的"数据处理入门"教程到此结束。

<a name="next-steps"> </a>
## 后续步骤

本教程演示了有关如何使 Android 应用程序处理移动服务中的数据的基础知识。

请试着学习下列教程之一：

-   [身份验证入门][]
    了解如何对应用程序用户进行身份验证。

-   [推送通知入门][]
    了解如何向应用程序发送一条非常简单的推送通知。

-   [移动服务 .NET 操作方法概念性参考][]
    了解有关如何将移动服务与 .NET 一起使用的详细信息。

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/ "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/ "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-data/ "Windows Phone"
  [Android]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started-data/ "Android"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started-data/ ".NET 后端"
  [JavaScript 后端]: /zh-cn/develop/mobile/tutorials/get-started-with-data-android/ "JavaScript 后端"
  [JavaScript 后端版本]: /zh-cn/develop/mobile/tutorials/get-started-with-data-android
  [创建新的移动服务]: #create-service
  [在本地下载服务]: #download-the-service-locally
  [测试移动服务]: #test-the-service
  [将移动服务发布到 Azure]: #publish-mobile-service
  [下载 GetStartedWithData 项目]: #download-app
  [更新应用程序以使用移动服务进行数据访问]: #update-app
  [针对发布的移动服务测试应用程序]: #test-app
  [Azure 免费试用]: http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=AE564AB28&returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fzh-cn%2Fdocumentation%2Farticles%2Fmobile-services-dotnet-backend-windows-store-dotnet-get-started-data%2F
  [mobile-services-dotnet-backend-create-new-service]: ../includes/mobile-services-dotnet-backend-create-new-service.md
  [mobile-services-download-service-locally]: ../includes/mobile-services-download-service-locally.md
  [mobile-services-dotnet-backend-test-local-service]: ../includes/mobile-services-dotnet-backend-test-local-service.md
  [mobile-services-dotnet-backend-publish-service]: ../includes/mobile-services-dotnet-backend-publish-service.md
  [1]: ../includes/download-android-sample-code.md
  [mobile-services-verify-android-sdk-version]: ../includes/mobile-services-verify-android-sdk-version.md
  [mobile-services-android-run-sample-code]: ../includes/mobile-services-android-run-sample-code.md
  [mobile-services-android-getting-started-with-data]: ../includes/mobile-services-android-getting-started-with-data.md
  [0]: ./media/mobile-services-dotnet-backend-android-get-started-data/manage-sql-azure-database.png
  [2]: ./media/mobile-services-dotnet-backend-android-get-started-data/sql-azure-query.png
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-android
  [推送通知入门]: /zh-cn/develop/mobile/tutorials/get-started-with-push-android
  [移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library
