<properties 
	pageTitle="使用移动服务中的软删除（Windows 应用商店）| Azure" 
	description="了解如何在你的应用程序中使用 Azure 移动服务软删除功能" 
	documentationCenter="" 
	authors="wesmc7777" 
	manager="dwrede" 
	editor="" 
	services="mobile-services"/>

<tags 
	ms.service="mobile-services" 
	ms.date="02/11/2016"
	wacn.date="03/21/2016"/>

#  使用移动服务中的软删除


##概述

使用 JavaScript 或.NET 后端创建的表可根据需要启用软删除。使用软删除时，将在数据库中添加一个名为 *__deleted* 的 [SQL 位类型]的新列。启用软删除后，删除操作不会以物理方式删除来自数据库的行，而是将已删除的列的值设置为 TRUE。

启用软删除后再查询表的记录时，默认情况下，已删除的行不会返回在查询中。若要请求这些行，必须在 [REST 查询操作](http://msdn.microsoft.com/zh-cn/library/azure/jj677199.aspx)中传递查询参数 *__includeDeleted=true*。在.NET 客户端 SDK 中，你还可以使用帮助器方法 `IMobileServiceTable.IncludeDeleted()`。

软删除支持首次与 1.0.402 版 Azure 移动服务.NET 后端发布的.NET 后端。可以从 [Azure 移动服务.NET 后端](http://go.microsoft.com/fwlink/?LinkId=513165)获取最新的 NuGet 包。


使用软删除的一些潜在优势：

* 使用[移动服务的脱机数据同步]时，客户端 SDK 会自动查询已删除的记录，并将其从本地数据库中删除。如果不启用软删除，你需要在后端编写附加代码，以便客户端 SDK 知道要从本地存储中清除哪些记录。否则，客户端本地存储和后端将在这些已删除的记录方面发生不一致，而你必须调用客户端方法 `PurgeAsync()` 来清除本地存储。
* 某些应用程序具有一项业务要求，即永远不以物理方式删除数据，或仅在已审核后删除数据。软删除功能在该场景中非常有用。
* 软删除可被用于实施“取消删除”功能，以便可以恢复意外删除的数据。
但是，软删除的记录会占用数据库中的空间，因此你应该考虑创建一个计划的作业定期硬删除软删除的记录。有关示例，请参阅[在 .NET 后端使用软删除](#using-with-dotnet)和[在 JavaScript 后端使用软删除](#using-with-javascript)。客户端代码也应定期调用 `PurgeAsync()`，以便这些硬删除的记录不会保留在设备的本地数据存储中。





## 启用面向.NET 后端的软删除

软删除支持首次与 1.0.402 版 Microsoft Azure 移动服务.NET 后端发布的.NET 后端。可以从 [Azure 移动服务.NET 后端](http://go.microsoft.com/fwlink/?LinkId=513165)获取最新的 NuGet 包。

以下步骤将指导你如何启用面向.NET 后端移动服务的软删除功能。

1. 在 Visual Studio 中打开 .NET 后端移动服务项目。
2. 右键单击.NET 后端项目，然后单击“管理 NuGet 包”。 
3. 在包管理器对话框中，单击更新下的 **Nuget.org**，然后安装 1.0.402 或更高版本的 [Azure 移动服务.NET 后端](http://go.microsoft.com/fwlink/?LinkId=513165) NuGet 包。
3. 在 Visual Studio 的解决方案资源管理器中，展开 .NET 后端项目下的“控制器”节点，然后打开控制器源代码。例如 *TodoItemController.cs*。
4. 在控制器的 `Initialize()` 方法中，将参数 `enableSoftDelete: true` 传递给 EntityDomainManager 构造函数。

        protected override void Initialize(HttpControllerContext controllerContext)
        {
            base.Initialize(controllerContext);
            MobileService1Context context = new MobileService1Context();
            DomainManager = new EntityDomainManager<TodoItem>(context, Request, Services, enableSoftDelete: true);
        }


## 启用面向 JavaScript 后端的软删除

如果你正在为你的移动服务创建一个新表，那么你可以在表创建页面上启用软删除。

![][2]

若要在 JavaScript 后端中的现有表上启用软删除：

1. 在 [Azure 管理门户]中单击你的移动服务。然后单击数据选项卡。
2. 在数据页面上，单击以选择所需的表。然后在命令栏中单击“启用软删除”按钮。如果表已启用软删除，则此按钮将不会出现，但当你单击表的“浏览”或“列”选项卡时，可以看到 *\_\_deleted* 列。

    ![][0]

    若要对表禁用软删除，请单击“列”选项卡，然后单击 *\_\_deleted* 列和“删除”按钮。

    ![][1]

##  <a name="using-with-dotnet"></a>在 .NET 后端使用软删除


以下计划的作业清除存在时间超过一个月的软删除的记录：

    public class SampleJob : ScheduledJob
    {
        private MobileService1Context context;
     
        protected override void Initialize(ScheduledJobDescriptor scheduledJobDescriptor, 
            CancellationToken cancellationToken)
        {
            base.Initialize(scheduledJobDescriptor, cancellationToken);
            context = new MobileService1Context();
        }
     
        public override Task ExecuteAsync()
        {
            Services.Log.Info("Purging old records");
            var monthAgo = DateTimeOffset.UtcNow.AddDays(-30);
     
            var toDelete = context.TodoItems.Where(x => x.Deleted == true && x.UpdatedAt <= monthAgo).ToArray();
            context.TodoItems.RemoveRange(toDelete);
            context.SaveChanges();
     
            return Task.FromResult(true);
        }
    }





## <a name="using-with-javascript"></a>在 JavaScript 后端使用软删除

你可以借助 JavaScript 后端移动服务，使用表脚本来添加关于软删除的逻辑。

若要检测“取消删除”的请求，请使用更新表脚本中的“取消删除”属性：
    
    function update(item, user, request) {
        if (request.undelete) { /* any undelete specific code */; }
    }
若要在脚本中包含查询结果内的已删除记录，请将“includeDeleted”参数设置为 true：
    
    tables.getTable('softdelete_scenarios').read({
        includeDeleted: true,
        success: function (results) {
            request.respond(200, results)
        }
    });

若要通过 HTTP 请求检索已删除的记录，请添加查询参数"\_\_includedeleted=true"：

    http://youservice.azure-mobile.net/tables/todoitem?__includedeleted=true

###  制定用于清除软删除记录的计划作业的样本。

这是一个计划作业示例，用于删除在某一特定日期之前更新的记录：

    function purgedeleted() {
         mssql.query('DELETE FROM softdelete WHERE __deleted=1', {
            success: function(results) {
                console.log(results);
            },
            error: function(err) {
                console.log("error is: " + err);
        }});
    }







<!-- Images -->

[0]: ./media/mobile-services-using-soft-delete/enable-soft-delete-button.png
[1]: ./media/mobile-services-using-soft-delete/disable-soft-delete.png
[2]: ./media/mobile-services-using-soft-delete/enable-soft-delete-new-table.png

<!-- URLs. -->

[SQL 位类型]: http://msdn.microsoft.com/zh-cn/library/ms177603.aspx
[移动服务的脱机数据同步]: /documentation/articles/mobile-services-windows-store-dotnet-get-started-offline-data/
[Azure 经典门户]: https://manage.windowsazure.cn/

<!---HONumber=Mooncake_0118_2016-->