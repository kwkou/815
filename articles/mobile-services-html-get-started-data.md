<properties linkid="develop-mobile-tutorials-get-started-with-data-html" urlDisplayName="数据处理入门 (HTML5)" pageTitle="数据处理入门 (HTML 5) | 移动开发人员中心" metaKeywords="" description="了解如何开始使用移动服务来利用 HTML 应用程序中的数据。" metaCanonical="" services="" documentationCenter="Mobile" title="Get started with data in Mobile Services" authors="glenga" solutions="" manager="" editor="" />



# 将移动服务添加到现有应用程序

[WACOM.INCLUDE [mobile-services-selector-get-started-data](../includes/mobile-services-selector-get-started-data.md)]

本主题说明如何通过 Azure 移动服务来利用 HTML 应用程序中的数据。在本教程中，你将下载一个可在内存中存储数据的应用程序，创建一个新的移动服务，将该移动服务与该应用程序相集成，然后登录到 Azure 管理门户以查看运行该应用程序时对数据所做的更改。

>[WACOM.NOTE]本教程旨在帮助你更好地了解如何使用移动服务通过 Azure 来存储数据以及从 HTML 应用程序检索数据。因此，本主题指导你完成的许多步骤已在移动服务快速入门中代你完成。如果这是你第一次体验移动服务，请考虑首先完成该教程 <a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-html">移动服务入门</a>。

本教程将指导你完成以下基本步骤：

1. [下载 HTML 应用程序项目]
2. [创建移动服务]
3. [添加用于存储的数据表]
4. [更新应用程序以使用移动服务]
5. [针对移动服务测试应用程序]

<div class="dev-callout"><strong>说明</strong> <p>若要完成本教程，您需要一个 Azure 帐户。如果你没有帐户，则可以创建一个免费的试用帐户，只需几分钟即可完成。有关详细信息，请参阅 <a href="http://www.windowsazure.cn/zh-cn/pricing/1rmb-trial/?WT.mc_id=A756A2826&amp;returnurl=http%3A%2F%2Fwww.windowsazure.cn%2Fzh-cn%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started-with-data-html%2F" target="_blank">Azure 免费试用</a>。</p></div> 

### 其他要求

你可以在任何 Web 服务器上托管 GetStartedWithData 应用程序。但是，为方便起见，我们提供了可让你在 `http://localhost:8000` 上运行应用程序的脚本。
 
+ 为了使用 localhost，本教程要求你在本地计算机上运行下列 Web 服务器之一：

	+  **在 Windows 上**：IIS Express。可通过 [Microsoft Web 平台安装程序]安装 IIS Express。   
	+  **在 MacOS X 上**：Python。该服务器事先应已安装。
	+  **在 Linux 上**：Python。必须安装[最新版本的 Python]。 
	
	你可以使用任何 Web 服务器来托管应用程序，但是这些 Web 服务器必须受下载的脚本支持。  

+ 支持 HTML5 的 Web 浏览器。

<h2><a name="download-app"></a>下载 GetStartedWithData 项目</h2>

本教程是在 [GetStartedWithData 应用程序]（一个 HTML5 应用程序）的基础上制作的。此应用程序的 UI 与移动服务快速入门中生成的应用程序相同，不过，前者的一些新增项本地存储在内存中。 

1. [下载 HTML 应用程序项目文件][GetStartedWithData 应用程序]。

2. 在 HTML 编辑器中打开下载的项目，然后检查 app.js 文件。

   	可以看到，新增的项已存储在内存中的 **Array** 对象 (**staticItems**) 内。刷新页面后这些数据将会消失，而不会持久保留。

3. 启动 **server** 子文件夹中的下列命令文件之一。

	+ **launch-windows**（Windows 计算机） 
	+ **launch-mac.command**（Mac OS X 计算机）
	+ **launch-linux.sh**（Linux 计算机）

	> [AZURE.NOTE] 在 Windows 计算机上，当 PowerShell 要求你确认是否要运行脚本时，请键入"R"。你的 Web 浏览器可能会警告你不要运行该脚本，因为它是从 Internet 下载的。如果出现此警告，你必须请求浏览器继续加载该脚本。
	
	随后将在本地计算机上启动用于托管新应用程序的 Web 服务器。

4. 在 Web 浏览器中打开 URL <a href="http://localhost:8000/" target="_blank">http://localhost:8000/</a> 以启动应用程序。

5. 在应用程序中的**"输入新任务"**中键入有意义的文本（例如 _Complete the tutorial_），然后单击**"添加"**。

   	![][0]  

   	可以看到，保存的文本已添加到 **staticItems** 数组，并且页面会刷新以显示新项。

<h2><a name="create-service"></a>在管理门户中创建新的移动服务</h2>

[WACOM.INCLUDE [mobile-services-create-new-service-data](../includes/mobile-services-create-new-service-data.md)]

<h2><a name="add-table"></a>将新表添加到移动服务</h2>

为了能够在新移动服务中存储应用数据，必须先在关联的 SQL Database 实例中创建一个新表。

1. 在管理门户中单击**"移动服务"**，然后单击您刚刚创建的移动服务。

2. 单击**"数据"**选项卡，然后单击**"+创建"**。

   	![][5]

   	此时将显示**"创建新表"**对话框。

3. 在**"表名"**中键入 _TodoItem_，然后单击勾选按钮。

  	![][6]

  	此时将创建一个设置了默认权限的新存储表 **TodoItem**。这意味着，具有应用程序密钥（随应用一起分发）的任何用户都可以访问和更改表中的数据。

    > [AZURE.NOTE] 移动服务快速入门中使用了相同的表名。但是，每个表是在特定于给定移动服务的架构中创建的。这是为了防止当多个移动服务使用同一数据库时发生数据冲突。

4. 单击新的 **TodoItem** 表，然后验证是否不存在任何数据行。

5. 单击**"列"**选项卡。验证系统是否已自动为您创建了以下默认列： 
	
	<table border="1" cellpadding="10">
 	<tr>
 	<th>列名</th>
 	<th>类型</th>
 	<th>索引</th>
 	</tr>
 	<tr>
 	<td>ID</td>
 	<td>字符串</td>
 	<td>已编制索引</td>
 	</tr>
 	<tr>
 	<td>__createdAt</td>
 	<td>日期</td>
 	<td>已编制索引</td>
 	</tr>
 	<tr>
 	<td>__updatedAt</td>
 	<td>日期</td>
 	<td><font color="transparent">-</font></td>
 	</tr>
 	<tr>
 	<td>__version</td>
 	<td>时间戳 (MSSQL)</td>
 	<td><font color="transparent">-</font></td>
 	</tr> 	
 	</table> 

  	这是对移动服务中的表的最低要求。 

    > [AZURE.NOTE] 如果在移动服务中启用了动态架构，则通过插入或更新操作向移动服务发送 JSON 对象时，将自动创建新列。

6. 在**"配置"**选项卡中，验证 `localhost` 是否已列在**"跨域资源共享(CORS)"**下的**"允许来自以下主机名的请求"**列表中。如果未列出，请在**"主机名"**字段中键入 `localhost`，然后单击**"保存"**。

  	![][11]

	> [AZURE.IMPORTANT] 如果将快速入门应用程序部署到除 localhost 以外的 Web 服务器，则必须将该 Web 服务器的主机名添加到**"允许来自以下主机名的请求"**列表中。有关详细信息，请参阅[跨域资源共享](http://msdn.microsoft.com/library/windowsazure/dn155871.aspx"%20target="_blank)。

现在，你可以将新移动服务用作应用程序的数据存储。

<h2><a name="update-app"></a>更新应用程序以使用移动服务进行数据访问</h2>

将移动服务准备就绪后，您可以更新应用，以便在移动服务而不是本地集合中存储项。 

3. 在管理门户中单击**"移动服务"**，然后单击您刚刚创建的移动服务。

4. 单击**"仪表板"**选项卡并记下**"站点 URL"**中的值，然后单击**"管理密钥"**并记下**"应用程序密钥"**中的值。

   	![][8]

  	从应用代码访问移动服务时，您需要使用这些值。

1. 在 Web 编辑器中打开 index.html 项目文件，然后将以下代码添加到页的脚本引用中：

        <script src='http://ajax.aspnetcdn.com/ajax/mobileservices/MobileServices.Web-1.1.2.min.js'></script>

5. 在编辑器中打开 app.js 文件，取消注释以下用于定义 **MobileServiceClient** 变量的代码，然后在 **MobileServiceClient** 构造函数中依次提供来自移动服务的 URL 和应用程序密钥：

	    var MobileServiceClient = WindowsAzure.MobileServiceClient,
			client = new MobileServiceClient('AppUrl', 'AppKey'),   		    

  	这将创建用于访问移动服务的 MobileServiceClient 的新实例。

6. 注释掉以下代码行：

		var itemCount = 0;
		var staticItems = [];

	数据将存储在移动服务中，而不是内存中的数组中。

6. 取消注释以下代码行：

        todoItemTable = client.getTable('todoitem');

   	此代码将为 SQL Database **TodoItem** 创建代理对象 (**todoItemTable**)。 

7. 将 **$('#add-item').submit** 事件处理程序替换为以下代码：

		$('#add-item').submit(function(evt) {
			var textbox = $('#new-item-text'),
				itemText = textbox.val();
			if (itemText !== '') {
				todoItemTable.insert({ text: itemText, complete: false })
					.then(refreshTodoItems);
			}
			textbox.val('').focus();
			evt.preventDefault();
		});


  	此代码将在表中插入一个新项。

8. 将 **refreshTodoItems** 方法替换为以下代码：

		function refreshTodoItems() {

			var query = todoItemTable;

			query.read().then(function(todoItems) {
				listItems = $.map(todoItems, function(item) {
					return $('<li>')
						.attr('data-todoitem-id', item.id)
						.append($('<button class="item-delete">Delete</button>'))
						.append($('<input type="checkbox" class="item-complete">').prop('checked', item.complete))
						.append($('<div>').append($('<input class="item-text">').val(item.text)));
				});
					   
				$('#todo-items').empty().append(listItems).toggle(listItems.length > 0);
				$('#summary').html('<strong>' + todoItems.length + '</strong> item(s)');
			});
		}
	   

   这将向移动服务发送一个返回所有项的查询。代码将循环访问结果，并在页上显示数据。 

9. 将 **$(document.body).on('change', '.item-text')** 和 **$(document.body).on('change', '.item-complete')** 事件处理程序替换为以下代码：
        
		$(document.body).on('change', '.item-text', function() {
			var newText = $(this).val();
			todoItemTable.update({ id: getTodoItemId(this), text: newText });
		});

		$(document.body).on('change', '.item-complete', function() {
			var isComplete = $(this).prop('checked');
			todoItemTable.update({ id: getTodoItemId(this), complete: isComplete })
				.then(refreshTodoItems);
		});
 
   	这样，每当更改文本或者选中复选框时，就会向移动服务发送项更新。

10. 将 **$(document.body).on('click', '.item-delete')** 事件处理程序替换为以下代码：

		$(document.body).on('click', '.item-delete', function () {
			todoItemTable.del({ id: getTodoItemId(this) }).then(refreshTodoItems);
		});

	这样，每当用户单击**"删除"**按钮时，就会向移动服务发送一个 delete 命令。

更新应用程序以使用移动服务作为后端存储后，便可以针对移动服务测试该应用程序。

<h2><a name="test-app"></a>针对新移动服务测试应用程序</h2>

4. 在 Web 浏览器中重新加载 URL <a href="http://localhost:8000/" target="_blank">http://localhost:8000/</a> 以启动应用程序。

    > [AZURE.NOTE] 如果需要重新启动 Web 服务器，请重复第一部分中的步骤。

2. 如前所述，在**"输入新任务"**中键入文本，然后单击**"添加"**。 

   	此时会将一个新项作为 insert 发送到移动服务。

3. 在[管理门户]中单击**"移动服务"**，然后单击你的移动服务。

4. 单击**"数据"**选项卡，然后单击**"浏览"**。

   	![][9]
  
   	可以看到，**TodoItem** 表现在包含了数据以及移动服务生成的 ID 值，并且已在该表中自动添加了列，以匹配应用程序中的 TodoItem 类。

5. 在应用程序中，选中列表中的某个项，然后返回到门户中的"浏览"选项卡并单击**"刷新"**。 

  	可以看到，整个值已从 **false** 更改为 **true**。

6. 在 app.js 项目文件中，找到 **RefreshTodoItems** 方法，并将定义 `query` 的代码行替换为以下代码：

   		var query = todoItemTable.where({ complete: false });

7. 再次加载页面，选中列表中的另一个项。

   	可以看到，选中的项现已从列表中消失。每次更新都会导致往返访问移动服务，随后返回筛选的数据。

**数据处理入门**教程到此结束。

## <a name="next-steps"> </a>后续步骤

本教程演示了有关如何使 HTML 应用程序处理移动服务中的数据的基础知识。接下来，建议你完成下列教程之一，这些教程是基于本教程中创建的 GetStartedWithData 应用程序制作的：

* [使用脚本验证和修改数据]
  <br/>了解更多有关使用移动服务中的服务器脚本验证和更改从应用程序发送的数据的信息。

* [使用分页优化查询]
  <br/>了解如何使用查询中的分页控制单个请求中处理的数据量。
 
完成数据系列教程后，请尝试完成[身份验证入门]中的其他某篇相关教程，以了解如何对应用程序的用户进行身份验证。

<!-- Anchors. -->
[下载 HTML 应用程序项目]: #download-app
[创建移动服务]: #create-service
[添加用于存储的数据表]: #add-table
[更新应用程序以使用移动服务]: #update-app
[针对移动服务测试应用程序]: #test-app
[后续步骤]:#next-steps

<!-- Images. -->
[0]: ./media/mobile-services-html-get-started-data/mobile-quickstart-startup-html.png




[5]: ./media/mobile-services-html-get-started-data/mobile-data-tab-empty.png
[6]: ./media/mobile-services-html-get-started-data/mobile-create-todoitem-table.png

[8]: ./media/mobile-services-html-get-started-data/mobile-dashboard-tab.png
[9]: ./media/mobile-services-html-get-started-data/mobile-todoitem-data-browse.png

[11]: ./media/mobile-services-html-get-started-data/mobile-services-set-cors-localhost.png

<!-- URLs. -->
[使用脚本验证和修改数据]: /zh-cn/documentation/articles/mobile-services-html-validate-modify-data-server-scripts
[使用分页优化查询]: /zh-cn/documentation/articles/mobile-services-html-add-paging-data
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-users-html

[Azure 管理门户]: https://manage.windowsazure.cn/
[管理门户]: https://manage.windowsazure.cn/
[GetStartedWithData 应用程序]:  http://go.microsoft.com/fwlink/?LinkID=286345

[移动服务 HTML/JavaScript 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-html-how-to-use-client-library

[跨域资源共享]: http://msdn.microsoft.com/library/windowsazure/dn155871.aspx

