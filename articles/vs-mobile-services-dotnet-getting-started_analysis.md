<properties title="Getting Started with Mobile Services" pageTitle="" metaKeywords="Azure, Getting Started, Mobile Services" description="" services="mobile-services" documentationCenter="" authors="ghogen, kempb" />

<tags ms.service="mobile-services" ms.workload="web" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="10/8/2014" ms.author="ghogen, kempb"></tags>

### 发生了什么情况？

###### 已添加引用

Azure 移动服务 NuGet 包已添加到您的项目。因此，下面的 .NET 引用也已添加到您的项目：Microsoft.WindowsAzure.Mobile, Microsoft.WindowsAzure.Mobile.Ext, Newtonsoft.Json, System.Net.Http.Extensions, System.Net.Http.Primitives

###### 移动服务的连接字符串值

在 App.xaml.cs 文件中，已使用选定移动服务的应用程序 URL 和应用程序密钥创建 MobileServiceClient 对象。

### 移动服务入门

###### 获取对表的引用

以下代码将获取对表（包含 TodoItem 数据）的引用，您可以将该引用用于后续操作以便读取和更新数据表。您将需要 TodoItem 类，并将其属性设置为解释移动服务为响应您的查询而发送的 JSON。

    public class TodoItem
    {
        public string Id { get; set; }

        [JsonProperty(PropertyName = "text")]
        public string Text { get; set; }

        [JsonProperty(PropertyName = "complete")]
        public bool Complete { get; set; }
    }

    IMobileServiceTable<TodoItem> todoTable = App.<yourClient>.GetTable<TodoItem>();

如果您的表权限已设置为**具有应用程序密钥的任何人**，则此代码有效。如果您更改权限以保护您的移动服务，您将需要添加用户身份验证支持。请参阅[身份验证入门][身份验证入门]。

###### 添加条目

将新的项目插入数据表。

    TodoItem todoItem = new TodoItem() { Text = "My first to do item", Complete = false };
    await todoTable.InsertAsync(todoItem);

###### 读取/查询表

以下代码将查询表以获取所有项目。请注意，它仅返回数据的第一页，默认为 50 个项目。因为它是一个可选的参数，您可以传入所需的页大小。

    List<TodoItem> items = await todoTable.ToListAsync();
    try
    {
        // Query that returns all items.   
        items = await todoTable.ToCollectionAsync();             
    }
    catch (MobileServiceInvalidOperationException e)
    {
        // handle exception
    }

###### 更新条目

更新数据表中的行。参数项是指要更新的 TodoItem 对象。

    await todoTable.UpdateAsync(item);

###### 删除条目

删除数据库中的行。参数项是指要删除的 TodoItem 对象。

    await todoTable.DeleteAsync(item);

  [身份验证入门]: http://azure.microsoft.com/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-users/
