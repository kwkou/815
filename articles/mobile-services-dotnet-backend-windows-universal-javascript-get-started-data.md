<properties urlDisplayName="Get Started with Data" pageTitle="数据处理入门（Windows 应用商店）| 移动开发人员中心" metaKeywords="" description="了解如何开始使用移动服务来利用 Windows 应用商店应用程序中的数据。" metaCanonical="" services="mobile-services" documentationCenter="Mobile" title="Get started with data in Mobile Services" authors="wesmc" solutions="" manager="dwrede" editor="" />


# 将移动服务添加到现有应用程序

[WACOM.INCLUDE [mobile-services-selector-get-started-data](../includes/mobile-services-selector-get-started-data.md)]

本主题说明如何使用 Azure 移动服务作为 Windows 应用商店应用程序的后端数据源。在本教程中，你将要为某个应用程序（该应用程序在内存中存储数据）下载一个 Visual Studio 2013 项目，创建一个新的移动服务，将该移动服务与该应用程序相集成，并查看运行该应用程序时对数据所做的更改。

你将在本教程中创建的移动服务是 .NET 后端移动服务。.NET 后端可让你使用 .NET 语言和 Visual Studio 实现移动服务中的服务器端业务逻辑，并且你可以在本地计算机上运行和调试移动服务。若要创建允许用 JavaScript 编写服务器端业务逻辑的移动服务，请参阅本主题的 JavaScript 后端版本。

>[AZURE.NOTE]本主题演示如何使用 Visual Studio Professional 2013 Update 3 中的工具，将新的移动服务连接到通用 Windows 应用程序。如果你无法升级到 Visual Studio Professional 2013 Update 3，或者你想要手动将移动服务项目添加到 Windows 应用商店应用程序解决方案，请参阅[此版本](/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-data)的主题。

本教程将指导你完成以下基本步骤：

1. [下载 Windows 应用商店应用程序项目]
2. [从 Visual Studio 创建新移动服务]
3. [在本地测试移动服务项目]
4. [更新应用程序以使用移动服务]
5. [将移动服务发布到 Azure]
6. [针对 Azure 中托管的服务测试应用程序]
7. [查看 SQL Database 中存储的数据]

若要完成本教程，你需要以下项：

* 有效的 Azure 帐户。如果你没有帐户，则可以创建一个免费的试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 免费试用](/zh-cn/pricing/1rmb-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%www.windowsazure.cn%2Fzh-cn%2Fdocumentation%2Farticles%2Fmobile-services-dotnet-backend-windows-universal-javascript-get-started-data%2F)。
* <a href="https://go.microsoft.com/fwLink/p/?LinkID=257546" target="_blank">Visual Studio Professional 2013</a>。可以使用免费试用版。

## <a name="download-app"></a>下载 GetStartedWithData 项目

[WACOM.INCLUDE [mobile-services-windows-universal-javascript-download-project](../includes/mobile-services-windows-universal-javascript-download-project.md)]

## <a name="create-service"></a>从 Visual Studio 创建新移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-create-new-service-vs2013](../includes/mobile-services-dotnet-backend-create-new-service-vs2013.md)]

<ol start="8"><li><p>在解决方案资源管理器中，导航到 <strong>services\mobileService\scripts</strong> 子文件夹，打开 service.js 脚本文件，并留意新的全局变量，如以下示例所示：</p> 

		<pre><code>var todolistClient = new WindowsAzure.MobileServiceClient(
                "https://todolist.azure-mobile.cn/",
		        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX");</code></pre>

	<p>此代码通过使用一个全局变量提供对应用程序中新移动服务的访问权限。客户端是通过提供新移动服务的 URI 和应用程序密钥来创建的。由于对此脚本的引用已添加到 default.html 文件中，因此该变量可用于也从此页引用的所有脚本文件。</p>
</li>
<li><p>打开 default.html 项目文件，找到对新的 service.js 脚本文件的引用并确保引用的路径如下所示：</p>
<pre><code>&lt;script src="/services/mobileServices/scripts/todolist.js"&gt;</script></code></pre>
<p>目前在 Visual Studio 中有一个 Bug，导致在路径中生成的文件夹名不正确。</p></li>
<li><p>右键单击 Windows Phone 应用程序项目，单击<strong>"添加"</strong>，单击<strong>"连接的服务..."</strong>，选择刚创建的移动服务，然后单击<strong>"确定"</strong>。 </p>
<p>此时，同一个新的代码文件将添加到 Windows Phone 应用商店应用程序项目中。请确保也修复添加到 default.html 文件中的引用路径。</p></li>
</ol>

现在，Windows 应用商店和 Windows Phone 应用商店应用程序均已连接到新移动服务。下一步是测试新的移动服务项目。

## <a name="test-the-service-locally"></a>在本地测试移动服务项目

[WACOM.INCLUDE [mobile-services-dotnet-backend-test-local-service-api-documentation](../includes/mobile-services-dotnet-backend-test-local-service-api-documentation.md)]

## <a name="update-app"></a>更新应用程序以使用移动服务

在本部分中，你将要更新通用 Windows 应用程序，以将移动服务用作应用程序的后端服务。只需对 GetStartedWithData.Shared 项目文件夹中的 default.js 项目文件进行更改。 

[WACOM.INCLUDE [mobile-services-windows-javascript-update-data-app](../includes/mobile-services-windows-javascript-update-data-app.md)]

## <a name="publish-mobile-service"></a>将移动服务发布到 Azure

[WACOM.INCLUDE [mobile-services-dotnet-backend-publish-service](../includes/mobile-services-dotnet-backend-publish-service.md)]

## <a name="test-azure-hosted"></a>测试 Azure 中托管的移动服务

现在我们可以针对 Azure 中托管的移动服务测试这两个版本的通用 Windows 应用程序。

[WACOM.INCLUDE [mobile-services-windows-universal-test-app](../includes/mobile-services-windows-universal-test-app.md)]

## <a name="view-stored-data"></a>查看 SQL Database 中存储的数据

[WACOM.INCLUDE [mobile-services-dotnet-backend-view-sql-data](../includes/mobile-services-dotnet-backend-view-sql-data.md)]

**数据处理入门**教程到此结束。

## <a name="next-steps"> </a>后续步骤

本教程演示了有关如何使通用 Windows 应用程序项目处理移动服务中的数据的基础知识。接下来，建议你完成下列教程之一，这些教程是基于本教程中创建的 GetStartedWithData 应用程序制作的：

* [使用脚本验证和修改数据]
  <br/>了解更多有关使用移动服务中的服务器脚本验证和更改从应用程序发送的数据的信息。

* [使用分页优化查询]
  <br/>了解如何使用查询中的分页控制单个请求中处理的数据量。

完成数据系列教程后，请试着学习下列其他教程之一：

* [身份验证入门]
  <br/>了解如何对应用程序的用户进行身份验证。

* [推送通知入门]
  <br/>了解如何将非常简单的推送通知发送到你的应用程序。

* [移动服务 .NET 操作方法概念性参考]
  <br/>了解有关如何将移动服务与 HTML 和 JavaScript 一起使用的详细信息。
  
<!-- Anchors. -->

[下载 Windows 应用商店应用程序项目]: #download-app
[从 Visual Studio 创建新移动服务]: #create-service
[在本地测试移动服务项目]: #test-the-service-locally
[更新应用程序以使用移动服务]: #update-app
[针对本地托管的服务测试应用程序]: #test-locally-hosted
[将移动服务发布到 Azure]: #publish-mobile-service
[针对 Azure 中托管的服务测试应用程序]: #test-azure-hosted
[查看 SQL Database 中存储的数据]: #view-stored-data
[后续步骤]:#next-steps

<!-- Images. -->
[0]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/app-view.png
[1]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/mobile-data-sample-download-javascript-vs13.png
[2]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/mobile-service-overview-page.png
[3]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/download-service-project.png
[4]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/add-service-project-to-solution.png
[5]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/download-publishing-profile.png
[6]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/add-existing-project-dialog.png
[7]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/vs-manage-nuget-packages.png
[8]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/manage-nuget-packages.png
[9]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/copy-mobileserviceclient-snippet.png
[10]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/vs-pasted-mobileserviceclient.png
[11]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/vs-build-solution.png
[12]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/vs-run-solution.png

[16]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/azure-items.png
[17]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/manage-sql-azure-database.png
[18]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/sql-azure-query.png
[19]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/vs-mobileservices-script-reference.png
[20]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/vs-build-service-project.png
[21]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/vs-start-debug-service-project.png
[22]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/service-welcome-page.png
[23]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/iis-express-tray.png

[26]: ./media/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/copy-service-and-packages-folder.png

<!-- URLs. -->
[使用脚本验证和修改数据]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-validate-modify-data-server-scripts
[使用分页优化查询]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-add-paging-data
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started/
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-users/
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-push/


[Azure 管理门户]: https://manage.windowsazure.cn/
[管理门户]: https://manage.windowsazure.cn/
[移动服务 SDK]: http://go.microsoft.com/fwlink/p/?LinkId=257545
[开发人员代码示例站点]:  http://go.microsoft.com/fwlink/p/?LinkID=510826
[移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-html-how-to-use-client-library/
[MobileServiceClient 类]: http://go.microsoft.com/fwlink/p/?LinkId=302030
  
