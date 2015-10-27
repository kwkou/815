<properties
	pageTitle="将移动服务添加到现有应用（Windows 应用商店 JavaScript）| Windows Azure"
	description="了解如何开始使用移动服务来利用 Windows 应用商店应用程序中的数据。"
	services="mobile-services"
	documentationCenter="windows"
	authors="ggailey777"
	manager="dwrede"
	editor=""/>

<tags 
	ms.service="mobile-services"
	ms.date="06/16/2015"
	wacn.date="10/22/2015"/>


# 将移动服务添加到现有应用程序

[AZURE.INCLUDE [mobile-services-selector-get-started-data-legacy](../includes/mobile-services-selector-get-started-data-legacy.md)]

##概述
此主题说明如何通过 Azure 移动服务来利用 Windows 应用商店应用程序中的数据。在本教程中，你将要下载一个可在内存中存储数据的 Visual Studio 2013 应用程序项目，创建一个新的移动服务，将该移动服务与该应用程序相集成，然后登录到 Azure 管理门户以查看运行该应用程序时对数据所做的更改。

若要完成本教程，您需要以下各项：

* 有效的 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial)。
* Visual Studio 2013，将使用它将 Windows 应用商店应用项目连接到移动服务。

##下载 GetStartedWithData 项目

本教程基于 [GetStartedWithMobileServices 应用][Developer Code Samples site]而编写，后者是 Visual Studio 2013 中的 Windows 应用商店应用项目。此应用的 UI 与移动服务快速入门生成的应用相同，不过，前者的一些新增项存储在本地内存中。

1. 从[开发人员代码示例站点]下载 GetStartedWithData 示例应用的 JavaScript 版本。 

2. 在 Visual Studio 中打开下载的项目，然后展开 **js** 文件夹并检查 default.js 文件。

   	请注意，添加的 **TodoItem** 对象已存储在内存中的 **List** 对象中。

3. 按 **F5** 键以重新构建项目并启动此应用。

4. 在应用的“插入 TodoItem”中键入一些文本，然后单击“保存”。

   	![][0]

   	可以看到，保存的文本已显示在“查询和更新数据”下的第二列中。

##<a name="create-service"></a>在 Visual Studio 中创建新的移动服务

[AZURE.INCLUDE [mobile-services-create-new-service-vs2013](../includes/mobile-services-create-new-service-vs2013.md)]

<ol start="7">
<li><p>在解决方案资源管理器中，展开 <i>services</i>、<i>mobile services</i>、<i><your_service></i> 文件夹，打开 service.js 脚本文件，并留意新的全局变量，如以下示例所示：</p> 

		<pre><code>var todolistClient = new WindowsAzure.MobileServiceClient(
                "https://todolist.azure-mobile.net/",
		        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX");</code></pre>

	<p>此代码通过使用一个全局变量提供对应用程序中新移动服务的访问权限。客户端是通过提供新移动服务的 URI 和应用程序密钥来创建的。由于对此脚本的引用已添加到 default.html 文件中，因此该变量可用于也从此页引用的所有脚本文件。</p>
</li>
</ol>

##添加数据存储的新表

[AZURE.INCLUDE [mobile-services-create-new-table-vs2013](../includes/mobile-services-create-new-table-vs2013.md)]

##更新应用程序以使用移动服务

[AZURE.INCLUDE [mobile-services-windows-javascript-update-data-app](../includes/mobile-services-windows-javascript-update-data-app.md)]

##针对新移动服务测试应用程序

1. 在 Visual Studio 中，按 F5 键运行应用程序。

2. 如前所述，在“插入 TodoItem”中键入文本，然后单击“保存”。

   	此时会将一个新项作为 insert 发送到移动服务。

3. 在[管理门户]中单击“移动服务”，然后单击你的移动服务。

4. 单击“数据”选项卡，然后单击“浏览”。

   	可以看到，**TodoItem** 表现在包含了数据以及移动服务生成的 ID 值，并且已在该表中自动添加了列，以匹配应用程序中的 TodoItem 类。

5. 在应用程序中，选中列表中的某个项，然后返回到门户中的“浏览”选项卡并单击“刷新”。

  	可以看到，整个值已从 **false** 更改为 **true**。

6. 在 default.js 项目文件中，将现有 **RefreshTodoItems** 函数替换为用于筛选出已完成项目的以下代码：

        var refreshTodoItems = function () {                     
            // More advanced query that filters out completed items. 
            todoTable.where({ complete: false })
               .read()
               .done(function (results) {
                   todoItems = new WinJS.Binding.List(results);
                   listItems.winControl.itemDataSource = todoItems.dataSource;
               });            
        };

7. 在应用中，选中列表中的另一个项目，然后单击“刷新”按钮。

   	可以看到，选中的项现已从列表中消失。每次刷新都会导致往返访问移动服务，随后返回筛选的数据。

**数据处理入门**教程到此结束。

## <a name="next-steps"></a>后续步骤

本教程演示了有关如何使 Windows 应用商店应用程序处理移动服务中的数据的基础知识。建议你接下来阅读下列其他主题之一：

* [向应用程序添加身份验证](/documentation/articles/mobile-services-windows-store-javascript-get-started-users)
  <br/>了解如何对应用程序用户进行身份验证。

* [向应用程序添加推送通知](/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-get-started-push)
  <br/>了解如何向应用程序发送一条很基本的推送通知。

* [移动服务 HTML/JavaScript 操作方法概念性参考](/documentation/articles/mobile-services-html-how-to-use-client-library)
  <br/>了解有关如何将移动服务与 HTML 和 JavaScript 一起使用的详细信息。

<!-- Anchors. -->

[Get the Windows Store app]: #download-app
[Create the mobile service]: #create-service
[Add a data table for storage]: #add-table
[Update the app to use Mobile Services]: #update-app
[Test the app against Mobile Services]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->
[0]: ./media/mobile-services-windows-store-javascript-get-started-data/mobile-quickstart-startup.png

[9]: ./media/mobile-services-windows-store-javascript-get-started-data-vs2013/mobile-todoitem-data-browse.png
[10]: ./media/mobile-services-windows-store-javascript-get-started-data-vs2013/mobile-data-sample-download-js-vs12.png


<!-- URLs. -->
[Azure Management Portal]: https://manage.windowsazure.cn/
[管理门户]: https://manage.windowsazure.cn/
[Mobile Services SDK]: http://go.microsoft.com/fwlink/?LinkId=257545
[Developer Code Samples site]: http://go.microsoft.com/fwlink/p/?LinkId=328660
[开发人员代码示例站点]: http://go.microsoft.com/fwlink/p/?LinkId=328660
[Mobile Services HTML/JavaScript How-to Conceptual Reference]: /documentation/articles/mobile-services-html-how-to-use-client-library

<!---HONumber=74-->