
## <a name="update-app"></a>更新应用以调用自定义 API

1. 在 Visual Studio 中，打开快速启动项目中的 default.html 文件，找到名为  `buttonRefresh` 的**按钮**元素，然后在其后添加以下新元素： 

		<button id="buttonCompleteAll" style="margin-left: 5px">Complete All</button>

	这样可将新按钮添加到页。 

2. 打开  `js` 项目文件夹中的 default.js 代码文件，找到 **refreshTodoItems** 函数，并确认该函数包含以下代码：

	    todoTable.where({ complete: false })
	       .read()
	       .done(function (results) {
	           todoItems = new WinJS.Binding.List(results);
	           listItems.winControl.itemDataSource = todoItems.dataSource;
	       });            

	这样可以筛选项，使得查询不返回已完成的项。

3. 在 **refreshTodoItems** 函数后，添加以下代码：

		var completeAllTodoItems = function () {
		    var okCommand = new Windows.UI.Popups.UICommand("OK");
		
		    // Asynchronously call the custom API using the POST method. 
		    mobileService.invokeApi("completeall", {
		        body: null,
		        method: "post"
		    }).done(function (results) {
		        var message = results.result.count + " item(s) marked as complete.";
		        var dialog = new Windows.UI.Popups.MessageDialog(message);
		        dialog.commands.append(okCommand);
		        dialog.showAsync().done(function () {
		            refreshTodoItems();
		        });
		    }, function (error) {
		        var dialog = new Windows.UI.Popups
		            .MessageDialog(error.message);
		        dialog.commands.append(okCommand);
		        dialog.showAsync().done();
		    });
		};

        buttonCompleteAll.addEventListener("click", function () {
            completeAllTodoItems();
        });

	此方法可处理新按钮的 **Click** 事件。在客户端上调用 **InvokeApiAsync** 方法，该客户端向新的自定义 API 发送 POST 请求。与任何错误相同，自定义 API 返回的结果也显示在消息对话框中。

## <a name="test-app"></a>测试应用

1. 在 Visual Studio 中，按 **F5** 键以重新生成项目并启动应用。

2. 在应用的"插入 TodoItem"中键入一些文本，然后单击"保存"。

3. 重复上述步骤，直至您将多个 ToDo 项添加到列表。

4. 单击"完成全部"按钮。

  	![](./media/mobile-services-windows-store-javascript-call-custom-api/mobile-custom-api-windows-store-completed.png)

	此时会显示一个消息框，指示标记为完成的多个项，并再次执行筛选查询，将所有项从列表中清除。
