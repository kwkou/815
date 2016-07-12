

既然需要经过身份验证才能访问 TodoItem 表中的数据，您可以使用移动服务分配的 userID 值来过滤返回的数据。

>[WACOM.NOTE]以下方法要求在"用户"的"授权级别"上应用"AuthorizeLevel"属性。这样可以仅允许经过身份验证的用户访问表。

1. 在 Visual Studio 2013 中，打开您的移动服务项目，展开 DataObjects 文件夹，然后打开 TodoItem.cs 项目文件。

	TodoItem 类定义数据对象，您需要添加用于筛选的 UserId 属性。

2. 将下面的新 UserId 属性添加到 **TodoItem** 类：

		public string UserId { get; set; }

	>[WACOM.NOTE] 使用默认数据库初始值设定项时，只要实体框架在代码优先模型定义中检测到数据模型更改，它就会删除并重新创建数据库。若要进行此数据模型更改并维护数据库中的现有数据，必须使用代码优先迁移。不能为 Azure 中的 SQL 数据库 使用默认的初始值设定项。有关更多信息，请参阅[如何使用代码优先迁移来更新数据模型](/zh-cn/documentation/articles/mobile-services-dotnet-backend-how-to-use-code-first-migrations/)。

3. 在"解决方案资源管理器"中，展开 Controllers 文件夹，打开 TodoItemController.cs 项目文件，然后添加下面的 **using** 语句：

		using Microsoft.WindowsAzure.Mobile.Service.Security;

	**TodoItemController** 类可为 TodoItem 表实现数据访问。 
 
4. 找到 **PostTodoItem** 方法，并在末尾的 **return** 语句之前添加以下代码：

		// Get the logged-in user.
	    var currentUser = User as ServiceUser;
	
	    // Set the user ID on the item.
	    item.UserId = currentUser.Id;

    在将 UserId 值插入 TodoItem 表之前，这段代码将 UserId 值添加到项，该值是经过身份验证的用户的用户 ID。 
	

5. 找到 **GetAllTodoItems** 方法，并将现有的 **return** 语句替换为以下代码行：

        // Get the logged-in user.
        var currentUser = User as ServiceUser;

        return Query().Where(todo => todo.UserId == currentUser.Id);

   	此查询可筛选返回的 TodoItem 对象，使得每个用户只收到他们插入的项。您可以选择 

6. 将移动服务项目重新发布到 Azure。
