<properties title="移动服务入门" pageTitle="" metaKeywords="Azure, Getting Started, Mobile Services" description="" services="mobile-services" documentationCenter="" authors="ghogen, kempb" />

<tags ms.service="mobile-services" ms.workload="web" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="10/8/2014" ms.author="ghogen, kempb"></tags>

> [AZURE.SELECTOR]
>
> -   [入门][入门]
> -   [发生了什么情况][发生了什么情况]

## 移动服务入门

为了跟踪这些代码，您需要首先执行的步骤取决于您连接的移动服务类型。

对于 JavaScript 后端移动服务，需创建一个名为“TodoItem”表。创建表的方法：在服务器资源管理器中的 Azure node 下定位移动服务，右键单击该移动服务的 Node 打开上下文菜单，然后选择**创建表**。输入“TodoItem”作为表名称。

如果您使用的是 .NET 后端移动服务，那么 Visual Studio 为您创建的默认项目模板中已经有一个 TodoItem 表，但您需要将其发布到 Azure。发布方法：在解决方案资源管理器中打开移动服务项目的上下文菜单，然后选择**发布 Web**。接受默认设置，然后选择**发布**。

##### 获取对表的引用

客户端对象已添加到您的项目。其名称是移动服务的名称，后面附加“Client”。以下代码将获取对表（包含 TodoItem 数据）的引用，您可以将该引用用于后续操作以便读取和更新数据表。

    var todoTable = yourMobileServiceClient.getTable('TodoItem');

##### 添加条目

将新的项目插入数据表。ID（类型字符串的 GUID）将自动创建为新行的主密钥。不要更改 ID 列的类型，因为移动服务基础结构将使用它。

    var todoTable = client.getTable('TodoItem');
    var todoItems = new WinJS.Binding.List();
    var insertTodoItem = function (todoItem) {
        todoTable.insert(todoItem).done(function (item) {
            todoItems.push(item);
        });
    };

##### 读取/查询表

以下代码将查询表以获取所有项目、更新本地集合并将结果绑定到 UI 元素 listItems。

        // This code refreshes the entries in the list view 
        // by querying the TodoItems table.
        todoTable.where()
            .read()
            .done(function (results) {
                todoItems = new WinJS.Binding.List(results);
                listItems.winControl.itemDataSource = todoItems.dataSource;
            });

您可以使用 where 方法来修改查询。下面是筛选出已完成项目的示例。

    todoTable.where(function () {
        return (this.complete === false && this.createdAt !== null);
    })
    .read()
    .done(function (results) {
        todoItems = new WinJS.Binding.List(results);
        listItems.winControl.itemDataSource = todoItems.dataSource;
    });

有关您可以使用的查询的更多示例，请参阅[查询对象][查询对象]。

##### 更新条目

更新数据表中的行。在此示例中，todoItem 是更新的项目，而且项目是从移动服务返回的相同项目。移动服务响应时，本地 todoItems 列表中的项目将使用 [splice][splice] 方法进行更新。对返回的 [Promise][Promise] 对象调用 [done][Promise] 方法以获取插入对象的副本并处理任何错误。

        todoTable.update(todoItem).done(function (item) {
            todoItems.splice(todoItems.indexOf(item), 1, item);
        });

##### 删除条目

删除数据表中的行。对返回的 [Promise][Promise] 对象调用 [done][Promise] 方法以获取插入对象的副本并处理任何错误。

    todoTable.delete(todoItem).done(function (item) {
        todoItems.splice(todoItems.indexOf(item), 1);
    }

[详细了解移动服务][详细了解移动服务]

  [入门]: /documentation/articles/vs-mobile-services-javascript-getting-started/
  [发生了什么情况]: /documentation/articles/vs-mobile-services-javascript-what-happened/
  [查询对象]: http://msdn.microsoft.com/library/azure/jj613353.aspx
  [splice]: http://msdn.microsoft.com/library/windows/apps/Hh700810.aspx
  [Promise]: 
  [详细了解移动服务]: http://azure.microsoft.com/documentation/services/mobile-services/
