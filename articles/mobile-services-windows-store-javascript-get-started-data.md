<properties linkid="develop-mobile-tutorials-get-started-with-data-js-vs2013" urlDisplayName="Get Started with Data" pageTitle="Get started with data (Windows Store JavaScript) | Mobile Dev Center" metaKeywords="" description="Learn how to get started using Mobile Services to leverage data in your Windows Store JavaScript app." metaCanonical="https://www.windowsazure.com/zh-cn/develop/mobile/tutorials/get-started-with-data-dotnet/" services="" documentationCenter="Mobile" title="Get started with data in Mobile Services" authors="glenga" solutions="" manager="" editor="" />

# 移动服务中的数据处理入门

<div class="dev-center-tutorial-selector sublanding"> 
	<a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-dotnet" title="Windows Store C#">Windows 应用商店 C#</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-js" title="Windows Store JavaScript" class="current">Windows 应用商店 JavaScript</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-wp8" title="Windows Phone">Windows Phone</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-ios" title="iOS">iOS</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-android" title="Android">Android</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-html" title="HTML">HTML</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-ios" title="Xamarin.iOS">Xamarin.iOS</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-android" title="Xamarin.Android">Xamarin.Android</a>
</div>	


<div class="dev-center-tutorial-subselector">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/" title=".NET backend">.NET 后端</a> | 
	<a href="/zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-data/"  title="JavaScript backend" class="current">JavaScript 后端</a>
</div>

<p>此主题说明如何通过 Azure 移动服务来利用 Windows 应用商店应用程序中的数据。在本教程中，你将要下载一个可在内存中存储数据的 Visual Studio 2013 应用程序项目，创建一个新的移动服务，将该移动服务与该应用程序相集成，然后登录到 Azure 管理门户以查看运行该应用程序时对数据所做的更改。</p>

<div class="dev-callout"><b>说明</b>

<p>本教程需要 Visual Studio 2013，使用它可以轻松地将 Windows 应用商店应用程序连接到移动服务。若要使用 Visual Studio 2012 完成同一基本过程，请按照<a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-js-vs2012/">使用 Visual Studio 2012 的移动服务中的数据处理入门</a>主题中的步骤进行操作。</p>
</div>

本教程将指导你完成以下基本步骤：

1.  [下载 Windows 应用商店应用程序项目][]
2.  [创建移动服务][]
3.  [添加用于存储的数据表][]
4.  [更新应用程序以使用移动服务][]
5.  [针对移动服务测试应用程序][]

<div class="dev-callout"><b>说明</b>

<p>若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=A756A2826&amp;returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started-with-data-js%2F" target="_blank">Azure 免费试用</a>。</p>
</div>

<a name="download-app"></a>
## 下载项目下载 GetStartedWithData 项目

本教程是在 [GetStartedWithMobileServices 应用程序][]（Visual Studio 2013 中的一个 Windows 应用商店应用程序项目）的基础上制作的。此应用程序的 UI 与移动服务快速入门中生成的应用程序相同，不过，前者的一些新增项本地存储在内存中。

1.  从[开发人员代码示例站点][GetStartedWithMobileServices 应用程序]下载 GetStartedWithData 示例应用程序的 JavaScript 版本。

    ![][0]

2.  在 Visual Studio 2012 Express for Windows 8 中打开下载的项目，然后展开 "js" 文件夹并检查 default.js 文件。

    请注意添加的 "TodoItem" 对象存储在内存中的 "List" 对象中。

3.  按 "F5" 键重新生成项目并启动应用程序。

4.  在应用程序的“插入 TodoItem”中键入一些文本 ，然后单击“保存” 。

    ![][1]

    可以看到，保存的文本已显示在“查询和更新数据”下的第二列中 。

<a name="create-service"></a>
## 创建移动服务从 Visual Studio 新建移动服务

[WACOM.INCLUDE [mobile-services-create-new-service-vs2013](../includes/mobile-services-create-new-service-vs2013.md)]

<ol start="7">
<li><p>在解决方案资源管理器中，展开 \*\*services\*\*、\*\*mobile services\*\*、\*\*\<your\_service\>\*\* 文件夹，打开 service.js 脚本文件，并留意新的全局变量，如以下示例所示：</p>

		<pre><code>var todolistClient = new WindowsAzure.MobileServiceClient(
                "https://todolist.azure-mobile.net/",
		        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX");</code></pre>

    <p>此代码通过使用一个全局变量提供对应用程序中新移动服务的访问权限。客户端是通过提供新移动服务的 URI 和应用程序密钥来创建的。由于对此脚本的引用已添加到 default.html 文件中，因此该变量可用于也从此页引用的所有脚本文件。</p>
</li>
</ol>

<a name="add-table"></a>
## 添加新表将新表添加到移动服务并更新应用程序

[WACOM.INCLUDE [mobile-services-create-new-table-vs2013](../includes/mobile-services-create-new-table-vs2013.md)]

1.  在 default.js 脚本文件中，注释用于定义现有项目集合的行，然后取消注释或添加以下代码行，并将 `<yourClient>;` 替换为在将项目连接到移动服务时添加到 service.js 文件中的变量：

        var todoTable = <yourClient>.getTable('TodoItem');

    此代码将为新的数据库表创建代理对象 ("todoTable")。

2.  将 "InsertTodoItem" 函数替换为以下代码：

        var insertTodoItem = function (todoItem) {
        // Inserts a new row into the database.When the operation completes
        // and Mobile Services has assigned an id, the item is added to the binding list.
        todoTable.insert(todoItem).done(function (item) {
        todoItems.push(item);
            });
        };

    此代码将在表中插入一个新项。

	<div class="dev-callout"><b>说明</b>

    <p>将只使用 Id 列创建新表。启用动态架构后，移动服务将基于插入或更新请求中的 JSON 对象自动生成新列。有关详细信息，请参阅<a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/jj193175.aspx">动态架构</a>。</p>
	</div>

3.  将 "RefreshTodoItems" 函数替换为以下代码：

        var refreshTodoItems = function () {
        // This code refreshes the entries in the list by querying the table. 
        todoTable.read().done(function (results) {
        todoItems = new WinJS.Binding.List(results);
        listItems.winControl.itemDataSource = todoItems.dataSource;
            });
        };

    这将设置 todoTable 中的项目集合的绑定，其中包含从移动服务返回的所有 "TodoItem" 对象。

4.  将 "UpdateCheckedTodoItem" 函数替换为以下代码：

        var updateCheckedTodoItem = function (todoItem) {
        // This code takes a freshly completed TodoItem and updates the database. 
        todoTable.update(todoItem);
        };

    这会将项目更新发送给移动服务。

更新应用程序以使用移动服务作为后端存储后，便可以针对移动服务测试该应用程序。

<a name="test-app"></a>
## 测试应用程序针对新的移动服务测试应用程序

1.  在 Visual Studio 中，按 F5 键运行应用程序。

2.  如前所述，在“插入 TodoItem” 中键入文本，然后单击“保存” 。

    此时会将一个新项作为 insert 发送到移动服务。

3.  在[管理门户][]中单击“移动服务”，然后单击你的移动服务 。

4.  单击“数据”选项卡，然后单击“浏览” 。

    ![][2]

    可以看到，"TodoItem" 表现在包含了数据以及移动服务生成的 ID 值，并且已在该表中自动添加了列，以匹配应用程序中的 TodoItem 类。

5.  在应用程序中，选中列表中的某个项，然后返回到门户中的“浏览”选项卡并单击“刷新” 。

    可以看到，整个值已从 "false" 更改为 "true"。

6.  在 default.js 项目文件中，将现有 "RefreshTodoItems" 函数替换为用于筛选出已完成项目的以下代码：

        var refreshTodoItems = function () {                     
        // More advanced query that filters out completed items. 
        todoTable.where({ complete:false })
        .read()
        .done(function (results) {
        todoItems = new WinJS.Binding.List(results);
        listItems.winControl.itemDataSource = todoItems.dataSource;
               });            
        };

7.  在应用程序中，选中列表中的另一个项目，然后单击“刷新” 按钮。

    可以看到，选中的项现已从列表中消失。每次刷新都会导致往返访问移动服务，随后返回筛选的数据。

"数据处理入门"教程到此结束。

<a name="next-steps"> </a>
## 后续步骤

本教程演示了有关如何使 Windows 应用商店应用程序处理移动服务中的数据的基础知识。接下来，建议你完成下列教程之一，这些教程是基于本教程中创建的 GetStartedWithData 应用程序制作的：

-   [使用脚本验证和修改数据][]
    了解更多有关使用移动服务中的服务器脚本验证和更改从应用程序发送的数据的信息。

-   [使用分页优化查询][]
    了解如何使用查询中的分页控制单个请求中处理的数据量。

完成数据系列教程后，请试着学习下列教程之一：

-   [身份验证入门][]
    了解如何对应用程序用户进行身份验证。

-   [推送通知入门][]
    了解如何向应用程序发送一条非常简单的推送通知。

-   [移动服务 HTML/JavaScript 操作方法概念性参考][]
    了解有关如何将移动服务与 HTML 和 JavaScript 一起使用的详细信息。

  [Windows 应用商店 C\#]: /zh-cn/develop/mobile/tutorials/get-started-with-data-dotnet "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/develop/mobile/tutorials/get-started-with-data-js "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/develop/mobile/tutorials/get-started-with-data-wp8 "Windows Phone"
  [iOS]: /zh-cn/develop/mobile/tutorials/get-started-with-data-ios "iOS"
  [Android]: /zh-cn/develop/mobile/tutorials/get-started-with-data-android "Android"
  [HTML]: /zh-cn/develop/mobile/tutorials/get-started-with-data-html "HTML"
  [Xamarin.iOS]: /zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-ios "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-android "Xamarin.Android"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-data/ ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-data/ "JavaScript 后端"
  [使用 Visual Studio 2012 的移动服务中的数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-js-vs2012/
  [下载 Windows 应用商店应用程序项目]: #download-app
  [创建移动服务]: #create-service
  [添加用于存储的数据表]: #add-table
  [更新应用程序以使用移动服务]: #update-app
  [针对移动服务测试应用程序]: #test-app
  [Azure 免费试用]: http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=A756A2826&returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started-with-data-js%2F
  [GetStartedWithMobileServices 应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=328660
  [0]: ./media/mobile-services-windows-store-javascript-get-started-data-vs2013/mobile-data-sample-download-js-vs12.png
  [1]: ./media/mobile-services-windows-store-javascript-get-started-data-vs2013/mobile-quickstart-startup.png
  [mobile-services-create-new-service-vs2013]: ../includes/mobile-services-create-new-service-vs2013.md
  [mobile-services-create-new-table-vs2013]: ../includes/mobile-services-create-new-table-vs2013.md
  [动态架构]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj193175.aspx
  [管理门户]: https://manage.windowsazure.cn/
  [2]: ./media/mobile-services-windows-store-javascript-get-started-data-vs2013/mobile-todoitem-data-browse.png
  [使用脚本验证和修改数据]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-js
  [使用分页优化查询]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-js
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-js
  [推送通知入门]: /zh-cn/develop/mobile/tutorials/get-started-with-push-js
  [移动服务 HTML/JavaScript 操作方法概念性参考]: /zh-cn/develop/mobile/how-to-guides/work-with-html-js-client/
