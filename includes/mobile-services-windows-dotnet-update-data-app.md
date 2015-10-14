
1. 在 MainPage.xaml.cs 文件中，添加或取消注释以下 using 语句： 

		using Microsoft.WindowsAzure.MobileServices;

2. 将 TodoItem 类定义替换为以下代码： 

	    public class TodoItem
	    {
	        public string Id { get; set; }
	
	        [Newtonsoft.Json.JsonProperty(PropertyName = "text")]  
	        public string Text { get; set; }
	
	        [Newtonsoft.Json.JsonProperty(PropertyName = "complete")]  
	        public bool Complete { get; set; }
	    }
	
	**JsonPropertyAttribute** 用于定义客户端类型中的属性名称与基础数据表中的列名之间的映射。

	>[AZURE.NOTE]在通用 Windows 应用项目中，在共享的 DataModel 文件夹中的单独代码文件内定义了 TodoItem 类。

3. 在 MainPage.xaml.cs 中，注释掉或删除用于定义现有项集合的行，然后取消注释或添加以下行，将 _&lt;yourClient&gt;_ 替换为将项目连接到移动服务时添加到 App.xaml.cs 文件的 `MobileServiceClient`字段： 

		private MobileServiceCollection<TodoItem, TodoItem> items;
		private IMobileServiceTable<TodoItem> todoTable = 
		    App.<yourClient>.GetTable<TodoItem>();
		  
	此代码将创建一个移动服务感知型绑定集合 (items) 和数据库表 (todoTable) 的代理类。 

4. 在 **InsertTodoItem** 方法中，删除设置 **TodoItem.Id** 属性的代码行，为该方法添加 **async** 修饰符，然后取消注释以下代码行：

		await todoTable.InsertAsync(todoItem);


	此代码在表中插入一个新项。 

5. 将 **RefreshTodoItems** 方法替换为以下代码： 

		private async void RefreshTodoItems()
        {
            MobileServiceInvalidOperationException exception = null;
            try
            {
                // Query that returns all items.   
                items = await todoTable.ToCollectionAsync();             
            }
            catch (MobileServiceInvalidOperationException e)
            {
                exception = e;
            }
            if (exception != null)
            {
                await new MessageDialog(exception.Message, "Error loading items").ShowAsync();
            }
            else
            {
                ListItems.ItemsSource = items;
                this.ButtonSave.IsEnabled = true;
            }    
        }

	这将设置到  `todoTable` 中项集合的绑定，其中包含从移动服务返回的所有 **TodoItem** 对象。如果执行该查询时发生问题，将生成一个消息框来显示这些错误。 

6. 在 **UpdateCheckedTodoItem** 方法中，为该方法添加 **async** 修饰符，然后取消注释以下代码行： 

		await todoTable.UpdateAsync(item);

	这会将项更新发送给移动服务。 

既然此应用已更新从而将移动服务用于后端存储，就可以针对移动服务测试该应用。

<!---HONumber=71-->