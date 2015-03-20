

1. 在 default.js 脚本文件中，就在定义 todoItems 列表的代码行下方，添加以下函数定义：
 
        // Add a filter that adds a header to prevent caching. This makes sure that the 
		// latest data is returned when the 'Refresh; button is clicked.        
        var noCachingFilter = function (request, next, callback) {
            if (request.type === 'GET' && !request.headers['If-Modified-Since']) {
                request.headers['If-Modified-Since'] = 'Mon, 27 Mar 1972 00:00:00 GMT';
            }
            next(request, callback);
        };

	这将定义一个过滤器函数，该函数添加 `If-Modified-Since`头以防止在客户端上缓存。
 
2. 接下来，取消注释或添加以下代码行，然后将 `<yourClient>` 替换为将项目连接到移动服务时添加到 service.js 文件的变量：

		var todoTable = <yourClient>.withFilter(noCachingFilter).getTable('TodoItem');

   	此代码使用缓存过滤器为新的数据库表创建一个代理对象 (**todoTable**）。 

3. 将 **InsertTodoItem** 函数替换为以下代码：

		var insertTodoItem = function (todoItem) {
		    // Inserts a new row into the database. When the operation completes
		    // and Mobile Services has assigned an id, the item is added to the binding list.
		    todoTable.insert(todoItem).done(function (item) {
		        todoItems.push(item);
		    });
		};

	此代码在表中插入一个新项。

3. 将 **RefreshTodoItems** 函数替换为以下代码：

        var refreshTodoItems = function () {
            // This code refreshes the entries in the list by querying the table.
            // Results are filtered to remove completed items.
            todoTable.where({ complete: false })
                .read().done(function (results) {
                todoItems = new WinJS.Binding.List(results);
                listItems.winControl.itemDataSource = todoItems.dataSource;
            });
        };

   	这将设置到 todoTable 中项集合的绑定，其中包含从移动服务返回的所有 **TodoItem** 对象。 

4. 将 **UpdateCheckedTodoItem** 函数替换为以下代码：
        
        var updateCheckedTodoItem = function (todoItem) {
            // This code takes a freshly completed TodoItem and updates the database. 
            todoTable.update(todoItem);
            // Remove the completed item from the filtered list.
            todoItems.splice(todoItems.indexOf(todoItem), 1);
        };

   	这会将项更新发送给移动服务。

既然此应用已更新从而将移动服务用于后端存储，就可以针对移动服务测试该应用。
