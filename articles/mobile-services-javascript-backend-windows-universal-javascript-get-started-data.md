<properties urlDisplayName="Get Started with Data" pageTitle="数据处理入门 (Windows Store JavaScript) |移动开发人员中心" metaKeywords="" description="了解如何通过移动服务来利用 Windows Store JavaScript 应用程序中的数据。" metaCanonical="https://www.windowsazure.cn/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-data/" services="mobile-services" documentationCenter="Mobile" title="Get started with data in Mobile Services" authors="glenga" solutions="" manager="dwrede" editor="" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-windows-store" ms.devlang="javascript" ms.topic="article" ms.date="09/26/2014" ms.author="glenga" />


# 将移动服务添加到现有应用程序
[WACOM.INCLUDE [mobile-services-selector-get-started-data](../includes/mobile-services-selector-get-started-data.md)]

此主题说明如何通过 Azure 移动服务来利用通用 Windows 应用程序中的数据。通用 Windows 应用程序解决方案包括面向 Windows 应用商店 8.1 和 Windows Phone 应用商店 8.1 应用程序的项目以及常见的共享项目。有关详细信息，请参阅[构建面向 Windows 和 Windows Phone 的通用 Windows 应用程序](http://msdn.microsoft.com/library/windows/apps/xaml/dn609832.aspx)。

在本教程中，你将要下载一个可在内存中存储数据的 Visual Studio 2013 应用程序项目，创建一个新的移动服务，将该移动服务与该应用程序相集成，然后登录到 Azure 管理门户以查看运行该应用程序时对数据所做的更改。

>[AZURE.NOTE]本主题演示如何在 Visual Studio Express 2013 Update 3 中使用此工具，以便将新的移动服务连接到通用的 Windows 应用程序。可以使用相同的步骤将移动服务连接到 Windows 应用商店或 Windows Phone 应用商店 8.1 应用程序。将移动服务连接到 Windows Phone 8.0 或 Windows Phone Silverlight 8.1 应用程序，请参阅[面向 Windows Phone 的数据处理入门](/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-data)。

本教程将指导您完成以下基本步骤：

1. [下载 Windows 应用商店应用程序项目][获取 Windows 应用商店应用程序] 
2. [创建移动服务]
3. [添加用于存储的数据表]
4. [更新应用程序以使用移动服务]
5. [针对移动服务测试应用程序]

若要完成本教程，您需要以下各项：

* 有效的 Azure 帐户。如果没有帐户，则可创建一个免费的试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 免费试用](/zh-cn/pricing/1rmb-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fwww.windowsazure.cn%2Fzh-cn%2Fdocumentation%2Farticles%2Fmobile-services-javascript-backend-windows-universal-javascript-get-started-data%2F)。
* <a href="https://go.microsoft.com/fwLink/p/?LinkID=257546" target="_blank">Visual Studio Express 2013 for Windows</a> （update 2 或更高版本）。 

##<a name="download-app"></a>下载 GetStartedWithData 项目

[WACOM.INCLUDE [mobile-services-windows-universal-javascript-download-project](../includes/mobile-services-windows-universal-dotnet-download-project.md)]

##<a name="create-service"></a>从 Visual Studio 创建新的移动服务

[WACOM.INCLUDE [mobile-services-create-new-service-vs2013](../includes/mobile-services-create-new-service-vs2013.md)]

<ol start="8">
<li><p>在解决方案资源管理器，导航到 <strong>services\mobileService\scripts</strong> 子文件夹，打开 service.js 脚本文件，并注意新的全局变量，其外观类似于下面的示例：</p> 

		<pre><code>var todolistClient = new WindowsAzure.MobileServiceClient(
                "https://todolist.azure-mobile.cn/",
		        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX");</code></pre>

	<p>此代码通过使用一个全局变量提供对应用程序中新移动服务的访问权限。客户端是通过提供新移动服务的 URI 和应用程序密钥来创建的。由于对此脚本的引用已添加到 default.html 文件中，因此该变量可用于也从此页引用的所有脚本文件。</p>
</li>
<li><p>打开 default.html 项目文件，找到对新的 service.js 脚本文件的引用并确保引用的路径如下所示：</p>
<pre><code>&lt;script src="/services/mobileServices/scripts/todolist.js"&gt;</script></code></pre>
<p>目前 Visual Studio 中有一个漏洞，将导致在路径中生成不正确的文件夹名称。</p></li>
<li><p>右键单击 Windows Phone 应用程序项目，单击 <strong>将</strong>，单击 <strong>"连接的服务..."</strong>，选择您刚创建的移动服务，然后单击 <strong>确定</strong>。 </p>
<p>同一个新的代码文件添加到 Windows Phone 应用商店应用程序项目。请确保修复添加到 default.html 文件的引用路径。</p></li>
</ol>

##<a name="add-table"></a>将新表添加到移动服务

[WACOM.INCLUDE [mobile-services-create-new-table-vs2013](../includes/mobile-services-create-new-table-vs2013.md)]

>[AZURE.NOTE]将使用 Id、__createdAt、__updatedAt 和 __version 列创建新表。启用动态架构后，移动服务将基于插入或更新请求中的 JSON 对象自动生成新列。有关详细信息，请参阅[动态架构](http://msdn.microsoft.com/library/windowsazure/jj193175.aspx)。

#<a name="update-app"></a>更新应用程序以使用移动服务

[WACOM.INCLUDE [mobile-services-windows-javascript-update-data-app](../includes/mobile-services-windows-javascript-update-data-app.md)]

##<a name="test-azure-hosted"></a>测试在 Azure 中托管的移动服务

现在我们可以根据在 Azure 中托管的移动服务，测试这两个版本的通用 Windows 应用程序。

[WACOM.INCLUDE [mobile-services-windows-universal-test-app](../includes/mobile-services-windows-universal-test-app.md)]

<ol start="4">
<li><p>在 <a href="https://manage.windowsazure.cn/" target="_blank">管理门户</a>，单击 <strong>移动服务</strong>，然后单击你的移动服务。<p></li>
<li><p>单击 <strong>数据</strong> 选项卡，然后单击 <strong>浏览</strong>。</p>
<p>请注意， <strong>TodoItem</strong> 表现在包含了数据以及移动服务生成的 ID 值，并且已在该表中自动添加了列，以匹配应用程序中的 TodoItem 类。</p></li>
</ol>

![](./media/mobile-services-javascript-backend-windows-universal-dotnet-get-started-data/mobile-todoitem-data-browse.png)
     	
"数据处理入门"教程到此结束。

## <a name="next-steps"> </a>后续步骤

本教程演示了有关如何使通用 Windows 应用程序处理移动服务中的数据的基础知识。接下来，建议您完成下列教程之一，这些教程是基于本教程中创建的 GetStartedWithData 应用程序制作的：

* [使用脚本验证和修改数据]
  <br/>了解有关在移动服务中使用服务器脚本验证和更改从应用程序发送的数据的详细信息。

* [使用分页优化查询]
  <br/>了解如何在查询中使用分页来控制单个请求中处理的数据量。

完成数据系列教程后，请试着学习下列教程之一：

* [身份验证入门]
  <br/>了解如何对应用程序的用户进行身份验证。

* [推送通知入门] 
  <br/>了解如何将非常简单的推送通知发送到您的应用程序。

* [移动服务 HTML/JavaScript 操作方法概念性参考]
  <br/>了解有关如何通过 HTML 和 JavaScript 使用移动服务的详细信息。

<!-- Anchors. -->

[获取 Windows 应用商店应用程序]: #download-app
[创建移动服务]: #create-service
[添加用于存储的数据表]: #add-table
[更新应用程序以使用移动服务]: #update-app
[针对移动服务测试应用程序]: #test-app
[后续步骤]:#next-steps

<!-- Images. -->
[0]: ./media/mobile-services-windows-store-javascript-get-started-data-vs2013/mobile-quickstart-startup.png

[9]: ./media/mobile-services-windows-store-javascript-get-started-data-vs2013/mobile-todoitem-data-browse.png
[10]: ./media/mobile-services-windows-store-javascript-get-started-data-vs2013/mobile-data-sample-download-js-vs12.png


<!-- URLs. -->
[使用脚本验证和修改数据]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-validate-modify-data-server-scripts/
[使用分页优化查询]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-add-paging-data/
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-get-started/
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-data/
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-users/
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-get-started-push/

[Azure 管理门户]: https://manage.windowsazure.cn/
[管理门户]: https://manage.windowsazure.cn/
[移动服务 SDK]: http://go.microsoft.com/fwlink/?LinkId=257545
[开发人员代码示例网站]:  http://go.microsoft.com/fwlink/p/?LinkID=510826
[移动服务 HTML/JavaScript 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-html-how-to-use-client-library/
