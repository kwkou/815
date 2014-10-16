<properties title="Getting Started with Mobile Services" pageTitle="" metaKeywords="Azure, Getting Started, Mobile Services" description="" services="mobile-services" documentationCenter="" authors="ghogen, kempb" />

<tags ms.service="mobile-services" ms.workload="web" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="10/8/2014" ms.author="ghogen, kempb"></tags>

### 发生了什么情况？

###### 已添加引用

Windows Azure 移动服务库已通过 MobileServices.js 文件形式添加到您的项目。

###### 移动服务的连接字符串值

在 services\\mobileServices\\settings 文件夹中，已生成具有 MobileServiceClient 的新 JavaScript (.js) 文件，其中包含选定的移动服务的应用程序 URL 和应用程序密钥。

### 移动服务入门

###### 获取对表的引用

客户端对象已添加到您的项目。其名称是移动服务的名称，后面附加“Client”。以下代码将获取对表（包含 TodoItem 数据）的引用，您可以将该引用用于后续操作以便读取和更新数据表。

    var todoTable = yourMobileServiceClient.getTable('TodoItem');

###### 添加条目

将新的项目插入数据表。ID（类型字符串的 GUID）将自动创建为新行的主密钥。不要更改 ID 列的类型，因为移动服务基础结构将使用它。

    var todoTable = client.getTable('TodoItem');
    var todoItems = new WinJS.Binding.List();
    var insertTodoItem = function (todoItem) {
        todoTable.insert(todoItem).done(function (item) {
            todoItems.push(item);
        });
    };

###### 读取/查询表

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

###### 更新条目

更新数据表中的行。在此示例中，todoItem 是更新的项目，而且项目是从移动服务返回的相同项目。移动服务响应时，本地 todoItems 列表中的项目将使用 [splice][splice] 方法进行更新。对返回的 [Promise][Promise] 对象调用 [done][Promise] 方法以获取插入对象的副本并处理任何错误。

        todoTable.update(todoItem).done(function (item) {
            todoItems.splice(todoItems.indexOf(item), 1, item);
        });

###### 删除条目

删除数据表中的行。对返回的 [Promise][Promise] 对象调用 [done][Promise] 方法以获取插入对象的副本并处理任何错误。

    todoTable.delete(todoItem).done(function (item) {
        todoItems.splice(todoItems.indexOf(item), 1);
    }

  [查询对象]: http://msdn.microsoft.com/library/azure/jj613353.aspx
  [splice]: http://msdn.microsoft.com/library/windows/apps/Hh700810.aspx
  [Promise]: ()
