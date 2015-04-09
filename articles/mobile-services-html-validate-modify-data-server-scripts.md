<properties linkid="develop-mobile-tutorials-validate-modify-and-augment-data-html" urlDisplayName="验证数据 - HTML5" pageTitle="用于验证和修改数据的用户服务器脚本 (HTML 5) | 移动开发人员中心" metaKeywords="" description="了解如何使用服务器脚本验证和修改从 HTML 应用程序发送的数据。" metaCanonical="" services="" documentationCenter="Mobile" title="Validate and modify data in Mobile Services by using server scripts" authors="glenga" solutions="" manager="" editor="" />



# 使用服务器脚本在移动服务中验证和修改数据 

[WACOM.INCLUDE [mobile-services-selector-validate-modify-data](../includes/mobile-services-selector-validate-modify-data.md)]

本主题说明如何在 Azure 移动服务中利用服务器脚本。你可以在移动服务中注册服务器脚本，然后使用这些脚本对所要插入和更新的数据执行各种操作，包括验证和数据修改。在本教程中，你将要定义并注册用于验证和修改数据的服务器脚本。由于服务器端脚本的行为往往会影响到客户端，因此你还要更新 HTML 应用程序以利用这些新行为。

本教程将指导你完成以下基本步骤：

1. [添加字符串长度验证]
2. [更新客户端以支持验证]
3. [在插入操作中添加时间戳]
4. [更新客户端以显示时间戳]

本教程以前一教程[数据处理入门]中的步骤和示例应用程序为基础。在开始本教程之前，必须先完成[数据处理入门]。  

## <a name="string-length-validation"></a>添加验证

验证用户提交的数据的长度总不失为一种良好做法。首先，你要注册一个脚本，用于验证发送到移动服务的字符串数据长度，并拒绝过长（在本例中为 10 个字符以上）的字符串。

1. 登录到 [Azure 管理门户]、单击**"移动服务"**，然后单击您的应用。 

   	![][0]

2. 单击**"数据"**选项卡，然后单击 **TodoItem** 表。

   	![][1]

3. 单击**"脚本"**，然后选择**"插入"**操作。

   	![][2]

4. 将现有脚本替换为以下函数，然后单击**"保存"**。

        function insert(item, user, request) {
            if (item.text.length > 10) {
                request.respond(statusCodes.BAD_REQUEST, {
                    error: "Text cannot exceed 10 characters"
                });
            } else {
                request.execute();
            }
        }

    此脚本将检查 **TodoItem.text** 属性的长度，如果该长度超过 10 个字符，则发送错误响应。如果未超过 10 个字符，将调用 **execute** 函数以完成插入。

    > [AZURE.TIP] 在**"脚本"**选项卡中，依次单击**"清除"**和**"保存"**可以删除某个已注册的脚本。	

## <a name="update-client-validation"></a>更新客户端

移动服务会验证数据和发送错误响应，而你则需要更新你的应用程序，使之能够处理验证后生成的错误响应。

1. 从你在完成教程[数据处理入门]后修改的项目的 **server** 子文件夹中运行下列命令文件之一。

	+ **launch-windows**（Windows 计算机） 
	+ **launch-mac.command**（Mac OS X 计算机）
	+ **launch-linux.sh**（Linux 计算机）

	> [AZURE.NOTE] 在 Windows 计算机上，当 PowerShell 要求你确认是否要运行脚本时，请键入"R"。你的 Web 浏览器可能会警告你不要运行该脚本，因为它是从 Internet 下载的。如果出现此警告，你必须请求浏览器继续加载该脚本。

	随后将在本地计算机上启动用于托管应用程序的 Web 服务器。

1. 	打开 app.js 文件，将 **$('#add-item').submit()** 事件处理程序替换为以下代码：

		$('#add-item').submit(function(evt) {
			var textbox = $('#new-item-text'),
				itemText = textbox.val();
			if (itemText !== '') {
				todoItemTable.insert({ text: itemText, complete: false })
					.then(refreshTodoItems, function(error){
					alert(JSON.parse(error.request.responseText).error);
				});
			}
			textbox.val('').focus();
			evt.preventDefault();
		});

2. 在 Web 浏览器中，导航到 <a href="http://localhost:8000/" target="_blank">http://localhost:8000/</a>，然后在**"添加新任务"**中键入文本并单击**"添加"**。

   	可以看到，操作失败，并且错误处理功能在对话框中显示了错误响应。

## <a name="add-timestamp"></a>添加时间戳

前面的任务验证了某个插入操作，并已接受或拒绝该操作。现在，你要使用一个服务器脚本来更新已插入的数据，该脚本将在插入对象之前向其添加一个时间戳属性。

> [AZURE.NOTE] 此处演示的 **createdAt** 时间戳属性目前是多余的。移动服务会自动为每个表创建一个 **__createdAt** 系统属性。

1. 在[管理门户]中的**"脚本"**选项卡上，将当前的 **Insert** 脚本替换为以下函数，然后单击**"保存"**。

        function insert(item, user, request) {
            if (item.text.length > 10) {
                request.respond(statusCodes.BAD_REQUEST, {
                    error: 'Text length must be under 10'
                });
            } else {
                item.createdAt = new Date();
                request.execute();
            }
        }

    在通过调用 **request**.**execute** 插入对象之前，此函数会将一个新的 **createdAt** 时间戳属性添加到该对象，从而扩展前面的 insert 脚本。 

    > [AZURE.IMPORTANT] 首次运行此 insert 脚本时，必须启用动态架构。启用动态架构后，移动服务在首次执行时会自动将 **createdAt** 列添加到 **TodoItem** 表。默认情况下，将为新的移动服务启用动态架构，发布应用程序之前应该禁用动态架构。

2. 在 Web 浏览器中重新加载页，在**"添加新任务"**中键入文本（少于 10 个字符），然后单击**"添加"**。

   	你会发现，新的时间戳并未显示在应用程序 UI 中。

3. 返回管理门户，在**todoitem**表中单击**"浏览"**选项卡。
   
   	可以看到，现在出现了一个**createdAt**列，并且插入的新项带有一个时间戳值。
  
接下来，需要更新应用程序以显示这个新列。

## <a name="update-client-timestamp"></a>再次更新客户端

移动服务客户端将忽略响应中的、无法序列化成已定义类型上的属性的所有数据。最后一步是更新客户端以显示这些新数据。

1. 在编辑器中打开 app.js 文件，将 **refreshTodoItems** 函数替换为以下代码：

		function refreshTodoItems() {
			var query = todoItemTable.where(function () {
                return (this.complete === false && this.createdAt !== null);
            });

			query.read().then(function(todoItems) {
				var listItems = $.map(todoItems, function(item) {
					return $('<li>')
						.attr('data-todoitem-id', item.id)
						.append($('<button class="item-delete">Delete</button>'))
						.append($('<input type="checkbox" class="item-complete">')
							.prop('checked', item.complete))
						.append($('<div>').append($('<input class="item-text">').val(item.text))
						.append($('<span class="timestamp">' 
							+ (item.createdAt && item.createdAt.toDateString() + ' '
							+ item.createdAt.toLocaleTimeString() || '') 
							+ '</span>')));

				});

				$('#todo-items').empty().append(listItems).toggle(listItems.length > 0);
				$('#summary').html('<strong>' + todoItems.length + '</strong> item(s)');
			});
		}

   	这将显示新 **createdAt** 属性的日期部分。 

2. 在编辑器中打开 style.css 文件，将 `item-text` 类中的样式替换为以下代码：

		.item-text { width: 70%; height: 26px; line-height: 24px; 
			border: 1px solid transparent; background-color: transparent; }
		.timestamp { width: 30%; height: 40px; font-size: .75em; }

	这将会调整文本框的大小，并设置新时间戳文本的样式。
	
6. 重新加载页。 	

   	可以看到，只显示了你在更新 insert 脚本后插入的项的时间戳。

7. 同样在 **refreshTodoItems** 函数中，将定义查询的代码行替换为以下代码：

         var query = todoItemTable.where(function () {
                return (this.complete === false && this.createdAt !== null);
            });

   	此函数将更新查询，以便同时筛选掉没有时间戳值的项。
	
8. 重新加载页。

   	可以看到，创建的不带时间戳值的所有项已从 UI 中消失。

现在，你已完成了这篇数据处理教程。

## <a name="next-steps"> </a>后续步骤

现在你已完成本教程，建议你继续学习数据系列中的最后一篇教程：[使用分页优化查询]。

有关详细信息，请参阅[使用服务器脚本]和[移动服务 HTML/JavaScript 操作方法概念性参考]


<!-- Anchors. -->
[添加字符串长度验证]: #string-length-validation
[更新客户端以支持验证]: #update-client-validation
[在插入操作中添加时间戳]: #add-timestamp
[更新客户端以显示时间戳]: #update-client-timestamp
[后续步骤]: #next-steps

<!-- Images. -->
[0]: ./media/mobile-services-html-validate-modify-data-server-scripts/mobile-services-selection.png
[1]: ./media/mobile-services-html-validate-modify-data-server-scripts/mobile-portal-data-tables.png
[2]: ./media/mobile-services-html-validate-modify-data-server-scripts/mobile-insert-script-users.png


<!-- URLs. -->
[使用服务器脚本]: /zh-cn/documentation/articles/mobile-services-how-to-use-server-scripts
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-html
<!--[Authorize users with scripts]: /zh-cn/develop/mobile/tutorials/authorize-users-html -->
[使用分页优化查询]: /zh-cn/documentation/articles/mobile-services-html-add-paging-data
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-data-html
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-users-html

[管理门户]: https://manage.windowsazure.cn/
[Azure 管理门户]: https://manage.windowsazure.cn/
[移动服务 HTML/JavaScript 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-html-how-to-use-client-library
