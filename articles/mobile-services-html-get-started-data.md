<properties 
	pageTitle="将移动服务添加到现有应用 (HTML 5) | Windows Azure" 
	description="了解如何开始在现有 HTML 应用程序中使用移动服务。" 
	services="mobile-services" 
	documentationCenter="" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="08/16/2015" 
	wacn.date="10/03/2015"/>

#  将移动服务添加到现有应用程序

[AZURE.INCLUDE [mobile-services-selector-get-started-data](../includes/mobile-services-selector-get-started-data.md)]

## 概述 

本主题说明如何通过 Azure 移动服务来利用 HTML 应用程序中的数据。在本教程中，你将要下载一个可在内存中存储数据的应用程序，创建一个新的移动服务，将该移动服务与该应用程序相集成，然后登录到 Azure 管理门户以查看运行该应用程序时对数据所做的更改。

>[AZURE.NOTE]本教程旨在帮助你更好地了解如何使用移动服务通过 Azure 来存储数据以及从 HTML 应用程序检索数据。因此，本主题指导你完成的许多步骤已在移动服务快速入门中代你完成。如果这是你第一次体验移动服务，请考虑首先完成[移动服务入门]教程。

> [AZURE.IMPORTANT]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial)。

### 其他要求

你可以在任何 Web 服务器上托管 GetStartedWithData 应用程序。但是，为方便起见，我们提供了可让你在 `http://localhost:8000` 上运行应用程序的脚本。
 
+ 为了使用 localhost，本教程要求你在本地计算机上运行下列 Web 服务器之一：

	+  **在 Windows 上**：IIS Express。可通过 [Microsoft Web 平台安装程序] 安装 IIS Express。   
	+  **在 MacOS X 上**：Python，该服务器事先应已安装。
	+  **在 Linux 上**：Python。必须安装 [最新版本的 Python]。 
	
	你可以使用任何 Web 服务器来托管应用程序，但是这些 Web 服务器必须受下载的脚本支持。

+ 支持 HTML5 的 Web 浏览器。

## <a name="download-app"></a>下载 GetStartedWithData 项目

本教程是在 [GetStartedWithData 应用程序]（一个 HTML5 应用程序）的基础上制作的。此应用的 UI 与移动服务快速入门生成的应用相同，不过，前者的一些新增项存储在本地内存中。

1. [下载 HTML 应用程序项目文件][GetStartedWithData app]。

2. 在 HTML 编辑器中打开下载的项目，然后检查 app.js 文件。

   	可以看到，新增的项已存储在内存中的 **Array** 对象 (**staticItems**) 内。刷新页面后这些数据将会消失，而不会持久保留。

3. 启动 **server** 子文件夹中的下列命令文件之一。

	+ **launch-windows**（Windows 计算机） 
	+ **launch-mac.command**（Mac OS X 计算机）
	+ **launch-linux.sh**（Linux 计算机）

	> [AZURE.NOTE]在 Windows 计算机上，当 PowerShell 要求你确认是否要运行脚本时，请键入 `R`。你的 Web 浏览器可能会警告你不要运行该脚本，因为它是从 Internet 下载的。如果出现此警告，你必须请求浏览器继续加载该脚本。
	
	随后将在本地计算机上启动用于托管新应用程序的 Web 服务器。

4. 在 Web 浏览器中打开 URL <a href="http://localhost:8000/" target="_blank">http://localhost:8000/</a> 以启动该应用程序。

5. 在应用程序中的“输入新任务”中键入有意义的文本（例如 _Complete the tutorial_），然后单击“添加”。

   	![][0]

   	可以看到，保存的文本已添加到 **staticItems** 数组，并且页面会刷新以显示新项。

## <a name="create-service"></a>在管理门户中创建新的移动服务

[AZURE.INCLUDE [mobile-services-create-new-service-data](../includes/mobile-services-create-new-service-data.md)]

## <a name="add-table"></a>将新表添加到移动服务

为了能够在新移动服务中存储应用数据，必须先在关联的 SQL 数据库实例中创建一个新表。

1. 在管理门户中单击“移动服务”，然后单击你刚刚创建的移动服务。

2. 单击“数据”选项卡，然后单击“+创建”。

   	此时将显示“创建新表”对话框。

3. 在“表名”中键入 _TodoItem_，然后单击勾选按钮。

  	此时将创建一个设置了默认权限的新存储表 **TodoItem**。这意味着，具有应用程序密钥（随应用一起分发）的任何用户都可以访问和更改表中的数据。该表是在特定于给定移动服务的架构中创建的。这是为了防止当多个移动服务使用同一数据库时发生数据冲突。

4. 单击新的 **TodoItem** 表，然后验证是否不存在任何数据行。

	>[AZURE.NOTE]将使用 Id、\_\_createdAt、\_\_updatedAt 和 \_\_version 列创建新表。启用动态架构后，移动服务将基于插入或更新请求中的 JSON 对象自动生成新列。有关详细信息，请参阅[动态架构](http://msdn.microsoft.com/zh-cn/library/windowsazure/jj193175.aspx)。

5. 在“配置”选项卡中，验证 是否已列在“跨域资源共享(CORS)”下的“允许来自以下主机名的请求”列表中`localhost`。如果未列出，请在“主机名”字段中键入 `localhost`，然后单击“保存”。


	> [AZURE.IMPORTANT]如果将快速入门应用程序部署到除 localhost 以外的 Web 服务器，则必须将该 Web 服务器的主机名添加到“允许来自主机名的请求”列表。有关详细信息，请参阅[跨域资源共享](http://msdn.microsoft.com/zh-cn/library/windowsazure/dn155871.aspx"%20target="_blank)。

现在，您可以将新移动服务用作应用的数据存储。

## <a name="update-app"></a>更新应用程序以使用移动服务进行数据访问

将移动服务准备就绪后，您可以更新应用，以便在移动服务而不是本地集合中存储项。

1. 在管理门户中单击“移动服务”，然后单击你刚刚创建的移动服务。

2. 单击“仪表板”选项卡并记下“移动服务 URL”中的值，然后单击“管理密钥”并记下“应用程序密钥”中的值。

  	从应用代码访问移动服务时，您需要使用这些值。

3. 在 Web 编辑器中打开 index.html 项目文件，然后将以下代码添加到页的脚本引用中：

        <script src="http://ajax.aspnetcdn.com/ajax/mobileservices/MobileServices.Web-1.2.5.min.js"></script>

4. 在编辑器中打开 app.js 文件，取消注释以下用于定义 **MobileServiceClient** 变量的代码，然后在 **MobileServiceClient** 构造函数中依次提供来自移动服务的 URL 和应用程序密钥：

	    var MobileServiceClient = WindowsAzure.MobileServiceClient,
			client = new MobileServiceClient('AppUrl', 'AppKey'),   		    

  	这将创建用于访问移动服务的 MobileServiceClient 的新实例。

5. 注释掉以下代码行：

		var itemCount = 0;
		var staticItems = [];

	数据将存储在移动服务中，而不是内存中的数组中。

6. 取消注释以下代码行：

        todoItemTable = client.getTable('todoitem');

   	此代码将为 SQL 数据库 **TodoItem** 创建代理对象 (**todoItemTable**)。

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


  	此代码在表中插入一个新项。

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

	这样，每当用户单击“删除”按钮时，就会向移动服务发送一个 delete 命令。

既然此应用已更新从而将移动服务用于后端存储，就可以针对移动服务测试该应用。

## <a name="test-app"></a>针对新移动服务测试应用程序

1. 在 Web 浏览器中重新加载 URL <a href="http://localhost:8000/" target="_blank">http://localhost:8000/</a> 以启动该应用程序。

    > [AZURE.NOTE]如果需要重新启动 Web 服务器，请重复第一部分中的步骤。

2. 如前所述，在“输入新任务”中键入文本，然后单击“添加”。

   	此时会将一个新项作为 insert 发送到移动服务。

3. 在[管理门户]中单击“移动服务”，然后单击你的移动服务。

4. 单击“数据”选项卡，然后单击“浏览”。

   	可以看到，**TodoItem** 表现在包含了数据以及移动服务生成的 ID 值，并且已在该表中自动添加了列，以匹配应用程序中的 TodoItem 类。

5. 在应用程序中，选中列表中的某个项，然后返回到门户中的“浏览”选项卡并单击“刷新”。

  	可以看到，整个值已从 **false** 更改为 **true**。

6. 在 app.js 项目文件中，找到 **RefreshTodoItems** 方法，并将定义 `query` 的代码行替换为以下代码：

   		var query = todoItemTable.where({ complete: false });

7. 再次加载页面，选中列表中的另一个项。

   	可以看到，选中的项现已从列表中消失。每次更新都会导致往返访问移动服务，随后返回筛选的数据。

**数据处理入门**教程到此结束。

##  <a name="next-steps"></a>后续步骤

本教程演示了有关如何使 HTML 应用程序处理移动服务中的数据的基础知识。接下来，请尝试完成[向应用程序添加身份验证]中的其他某篇相关教程，以了解如何对应用程序的用户进行身份验证。如果你正在使用 Cordova 或 PhoneGap 应用程序项目，可以在[使用 Microsoft Azure 将通知推送到 Cordova 应用程序](https://msdn.microsoft.com/magazine/dn879353.aspx)中了解有关推送通知的详细信息。

<!-- Anchors. -->

[Download the HTML app project]: #download-app
[Create the mobile service]: #create-service
[Add a data table for storage]: #add-table
[Update the app to use Mobile Services]: #update-app
[Test the app against Mobile Services]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->

[0]: ./media/mobile-services-html-get-started-data/mobile-quickstart-startup-html.png

<!-- URLs. -->
[移动服务入门]: /documentation/articles/mobile-services-html-get-started
[向应用程序添加身份验证]: /documentation/articles/mobile-services-html-get-started-users
[Azure Management Portal]: https://manage.windowsazure.cn/
[管理门户]: https://manage.windowsazure.cn/
[GetStartedWithData app]: http://go.microsoft.com/fwlink/?LinkID=286345
[GetStartedWithData 应用程序]: http://go.microsoft.com/fwlink/?LinkID=286345

[Mobile Services HTML/JavaScript How-to Conceptual Reference]: /documentation/articles/mobile-services-html-how-to-use-client-library
[Cross-origin resource sharing]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn155871.aspx

<!---HONumber=71-->