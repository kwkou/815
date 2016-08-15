<properties
	pageTitle="开始使用 Visual Studio .NET 移动服务项目（连接服务）| Microsoft Azure"
	description="如何在 Visual Studio .NET 项目中开始使用 Azure 移动服务"
	services="mobile-services"
	documentationCenter=""
	authors="mlhoop"
	manager="douge"
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="01/05/2016"
	wacn.date="03/21/2016"/>

#  移动服务入门（.NET 项目）


为了跟踪这些代码，您需要首先执行的步骤取决于您连接的移动服务类型。

- 对于 JavaScript 后端移动服务，需创建一个名为“TodoItem”表。创建表的方法：在服务器资源管理器中的 Azure node 下定位移动服务，右键单击该移动服务的 Node 打开上下文菜单，然后选择**创建表**。输入“TodoItem”作为表名称。

- 如果你使用的是 .NET 后端移动服务，那么 Visual Studio 为你创建的默认项目模板中已经有一个 TodoItem 表，但你需要将其发布到 Azure。发布方法：在解决方案资源管理器中打开移动服务项目的上下文菜单，然后选择**发布 Web**。接受默认设置，然后选择**发布**。

##获取对表的引用

以下代码将创建对表（`todoTable`，其中包含 TodoItem 的数据）的引用，你可以将该引用用于后续操作以便读取和更新数据表。您将需要 TodoItem 类，并将其属性设置为解释移动服务为响应您的查询而发送的 JSON。

	public class TodoItem
    {
        public string Id { get; set; }

        [JsonProperty(PropertyName = "text")]
        public string Text { get; set; }

        [JsonProperty(PropertyName = "complete")]
        public bool Complete { get; set; }
    }

	IMobileServiceTable<TodoItem> todoTable = App.<yourClient>.GetTable<TodoItem>();

如果您的表权限已设置为**具有应用程序密钥的任何人**，则此代码有效。如果您更改权限以保护您的移动服务，您将需要添加用户身份验证支持。请参阅[身份验证入门](/documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-users/)。

##添加表项

将新的项目插入数据表。

	TodoItem todoItem = new TodoItem() { Text = "My first to do item", Complete = false };
	await todoTable.InsertAsync(todoItem);

##读取或查询表

以下代码将查询表以获取所有项目。请注意，它仅返回数据的第一页，默认为 50 个项目。因为它是一个可选的参数，您可以传入所需的页大小。

    List<TodoItem> items;
    try
    {
        // Query that returns all items.   
        items = await todoTable.ToListAsync();             
    }
    catch (MobileServiceInvalidOperationException e)
    {
        // handle exception
    }


##更新表项

更新数据表中的行。参数项是指要更新的 TodoItem 对象。

	await todoTable.UpdateAsync(item);

##删除表项

删除数据库中的行。参数项是指要删除的 TodoItem 对象。

	await todoTable.DeleteAsync(item);


[详细了解移动服务](/documentation/services/mobile-services)

<!---HONumber=Mooncake_0215_2016-->