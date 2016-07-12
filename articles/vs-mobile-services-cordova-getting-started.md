<properties
	pageTitle="Cordova 移动服务项目（Visual Studio 连接服务）入门 | Microsoft Azure"
	description="描述使用 Visual Studio 连接服务将 Cordova 项目连接到 Azure 移动服务后可以执行的前几个步骤。"
	services="mobile-services"
	documentationCenter=""
	authors="mlhoop"
	manager="douge"
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="01/05/2016"
	wacn.date="03/21/2016"/>

#  移动服务入门（Cordova 项目）


##前几个步骤
为了跟踪这些代码，您需要首先执行的步骤取决于您连接的移动服务类型。

- 对于 JavaScript 后端移动服务，需创建一个名为“TodoItem”表。创建表的方法：在服务器资源管理器中的 Azure node 下定位移动服务，右键单击该移动服务的 Node 打开上下文菜单，然后选择**创建表**。输入“TodoItem”作为表名称。

- 如果你使用的是 .NET 后端移动服务，那么 Visual Studio 为你创建的默认项目模板中已经有一个 TodoItem 表，但你需要将其发布到 Azure。发布方法：在解决方案资源管理器中打开移动服务项目的上下文菜单，然后选择**发布 Web**。接受默认设置，然后选择**发布**。



##创建对表的引用

以下代码将获取对表（包含 TodoItem 数据）的引用，您可以将该引用用于后续操作以便读取和更新数据表。在您创建移动服务时，将自动创建 TodoItem 表。

var todoTable = mobileServiceClient.getTable('TodoItem');

要使这些示例工作，表的权限必须设置为**具有应用程序密钥的任何人**。稍后，您可以设置身份验证。请参阅[身份验证入门](/documentation/articles/mobile-services-html-get-started-users/)。

##向表中添加项

将新的项目插入数据表。ID（类型字符串的 GUID）将自动创建为新行的主密钥。对返回的 [Promise](https://msdn.microsoft.com/zh-cn/library/dn802826.aspx) 对象调用 **done** 方法以获取插入对象的副本并处理任何错误。

    function TodoItem(text) {
        this.text = text;
        this.complete = false;
    }
    
    var items = new Array();
    var insertTodoItem = function (todoItem) {
        todoTable.insert(todoItem).done(function (item) {
            items.push(item)
        });
    };

##读取或查询表

以下代码将查询表以获取所有项目，按文本字段排序。您可以添加代码以处理 success 处理程序中的查询结果。在此情况下，将更新这些项目的本地数组。

    todoTable.orderBy('text')
        .read().done(function (results) {
            items = results.slice();
        });

您可以使用 where 方法来修改查询。下面是筛选出已完成项目的示例。

    todoTable.where(function () {
            return (this.complete === false);
        })
        .read().done(function (results) {
            items = results.slice();
        });

有关您可以使用的查询的更多示例，请参阅[查询](http://msdn.microsoft.com/zh-cn/library/azure/jj613353.aspx)对象。

##更新表项

更新数据表中的行。在此代码中，当移动服务响应时，将从列表中删除项目。对返回的 [Promise](https://msdn.microsoft.com/zh-cn/library/dn802826.aspx) 对象调用 **done** 方法以获取插入对象的副本并处理任何错误。

    todoTable.update(todoItem).done(function (item) {
        // Update a local collection of items.
        items.splice(items.indexOf(todoItem), 1, item);
    });

##删除表项

使用 **del** 方法删除数据表中的行。对返回的 [Promise](https://msdn.microsoft.com/zh-cn/library/dn802826.aspx) 对象调用 **done** 方法以获取插入对象的副本并处理任何错误。

    todoTable.del(todoItem).done(function (item) {
        items.splice(items.indexOf(todoItem), 1);
    });

[详细了解移动服务](/documentation/services/mobile-services)

<!---HONumber=Mooncake_0215_2016-->