<properties 
	pageTitle="如何对 .NET 后端移动服务进行数据模型更改" 
	description="本主题介绍数据模型初始值设定项，以及如何在 .NET 后端移动服务进行数据模型更改。" 
	services="mobile-services" 
	documentationCenter="" 
	authors="ggailey777" 
	writer="glenga" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="02/07/2016"
	wacn.date="03/28/2016"/>

# 如何对 .NET 后端移动服务进行数据模型更改


本主题说明如何使用 Entity Framework 中的 Code First 迁移对现有的 Azure SQL 数据库进行数据模型更改，以避免丢失现有数据。此过程假设你已将 .NET 后端项目发布到 Azure，数据库中已有数据，并且远程和本地数据模型仍然保持同步。本主题还将介绍 Azure 移动服务实现的、在开发期间使用的默认 Code First 初始值设定项。这些初始值设定项可让你在无需维护现有数据的情况下，不使用 Code First 迁移就能轻松进行架构更改。

>[AZURE.NOTE]用作 SQL 数据库中表的前缀的架构名称在 web.config 文件内的 MS\_MobileServiceName 应用设置中定义。当你从门户下载初学者项目后，此值已设置为移动服务名称。如果架构名称与移动服务匹配，则多个移动服务可以安全共享同一个数据库实例。

请注意，.NET 后端项目不支持自动迁移。

## 更新数据模型

当你在 .NET 后端移动服务中添加功能时，会添加新的控制器来公开 API 中的新终结点。可以将新的 API 创建为自定义控制器或表控制器。[TableController<TEntity>] 公开继承自 [EntityData] 的数据类型。要使数据持久保存在数据库中，还必须在 [DbContext] 中将此数据类型作为新的 [DbSet<T>] 添加到数据模型中。若要了解有关 Entity Framework 中 Code First 的详细信息，请参阅[使用 Code First 创建模型](https://msdn.microsoft.com/zh-cn/data/ee712907#codefirst)。

使用 Visual Studio 可以轻松地创建用于向客户端应用公开新数据类型的新表控制器。有关详细信息，请参阅[如何使用控制器访问移动服务中的数据](https://msdn.microsoft.com/zh-cn/library/windows/apps/xaml/dn614132.aspx)。

## 数据模型初始值设定项

移动服务在 .NET 后端移动服务项目中提供两个数据模型初始值设定项基类。当 Entity Framework 在 [DbContext] 中检测到数据模型更改时，这两个初始值设定项将删除然后重新创建数据库中的表。不管移动服务在本地计算机上运行，还是在 Azure 中托管，都可以使用这些初始值设定项。

>[AZURE.NOTE]当你发布 .NET 后端移动服务时，在发生数据访问操作之前，初始值设定项将不会运行。这意味着，对于新发布的服务而言，只有在客户端请求数据访问操作（例如查询）之后，才会创建存储的表。
>
>你也可以使用内置的 API 帮助功能（从启动页上的“试用”链接访问）执行数据访问操作。有关使用 API 页面测试移动服务的详细信息，请参阅[将移动服务添加到现有应用程序](/documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-data/#test-the-service-locally)中的“在本地测试移动服务项目”部分。

这两个初始值设定项会从数据库中，删除移动服务所用架构中的所有表、视图、函数和过程。

+ **ClearDatabaseSchemaIfModelChanges** <br/>仅当 Code First 检测到数据模型更改时，才删除架构对象。从 [Azure 管理门户]下载的 .NET 后端项目中的默认初始值设定项继承自这个基类。
 
+ **ClearDatabaseSchemaAlways**：<br/>每次访问数据模型都会删除架构对象。使用此基类可以重置数据库，而无需进行数据模型更改。

在本地计算机上运行时，可以使用其他 Code First 数据模型初始值设定项。但是，由于用户没有删除数据库的权限，因此尝试删除数据库的初始值设定项在 Azure 中会失败，这是件好事。

在本地开发移动服务期间，你可以持续使用初始值设定项，而 .NET 后端教程也假设你使用的是初始值设定项。但是，如果你要做出数据模型更改并在数据库中维护现有数据，则必须使用 Code First 迁移。

>[AZURE.IMPORTANT]针对实时 Azure 服务开发和测试移动服务项目时，始终应该使用专门用于测试的移动服务实例。切勿针对当前已投入生产或者正由客户端应用程序使用的移动服务开发或测试项目。

在下载的快速入门项目中，Code First 初始值设定项在 WebApiConfig.cs 文件中定义。重写 **Seed** 方法可将开头几行数据添加到新表中。有关设定数据种子的示例，请参阅[在迁移中设定数据种子]。

## <a name="migrations"></a>启用 Code First 迁移

Code First 迁移使用快照方法来生成代码，执行这些代码会对数据库进行架构更改。借助迁移，你可对数据模型进行增量更改并在数据库中维护现有数据。

>[AZURE.NOTE]如果你已将 .NET 后端移动服务项目发布到 Azure，而你的 SQL 数据库表架构与项目的当前数据模型不匹配，则你必须使用初始值设定项并手动删除表，或者使架构和数据模型保持同步，然后尝试使用 Code First 迁移进行发布。

以下步骤将启用迁移，并应用项目、本地数据库和 Azure 中的数据模型更改。

1. 在 Visual Studio 的解决方案资源管理器中，右键单击移动服务项目，然后单击“设为启动项目”。
 
2. 在“工具”菜单中，展开“NuGet Package Manager”，然后单击“Package Manager Console”。

	此时会显示 Package Manager Console，你可以使用它来管理 Code First 迁移。

3. 在 Package Manager Console 中运行以下命令：

		PM> Enable-Migrations

	这将为你的项目启用代码优先迁移。

4. 在控制台中运行以下命令：

		PM> Add-Migration Initial

	这将创建一个名为 *Initial* 的新迁移。迁移代码存储在迁移项目文件夹中。

5. 展开 App\_Start 文件夹，打开 WebApiConfig.cs 项目文件并添加以下 **using** 语句：

		using System.Data.Entity.Migrations;
		using todolistService.Migrations;

	在上述代码中，必须将 _todolistService_ 字符串替换为项目的命名空间，对于下载的快速入门项目，该命名空间为 <em>mobile&#95;service&#95;name</em>Service。
 
6. 在这同一个代码文件中，注释掉对 **Database.SetInitializer** 方法的调用，并在该调用的后面添加以下代码：

        var migrator = new DbMigrator(new Configuration());
        migrator.Update();

	这样就会禁用用于删除然后重新创建数据库的默认 Code First 数据库初始值设定项，并将其替换为应用最新迁移的显式请求。此时，除非为数据创建了迁移，否则，在访问数据时，进行任何数据模型更改都会导致 InvalidOperationException。另外，你的服务必须使用 Code First 迁移将数据模型更改迁移到数据库。

7.  按 F5 在本地计算机上启动移动服务项目。
 
	此时，数据库已与数据模型同步。如果你提供了种子数据，可以通过依次单击“试用”、“GET 表/todoitem”、“试用此项”和“发送”来验证该数据。有关详细信息，请参阅[在迁移中设定数据种子]。

8.   现在，对数据模型进行更改（例如，向 TodoItem 类型添加一个新的 UserId 属性），重新生成项目，然后在 Package Manager 中运行以下命令：

		PM> Add-Migration NewUserId
                                                               
	这将创建一个名为 *NewUserId* 的新迁移。迁移文件夹中添加了一个用于实施此更改的新代码文件

9.  再次按 F5 以在本地计算机上重新启动移动服务项目。

	迁移已应用到数据库，并且数据库已重新与数据模型同步。如果你提供了种子数据，可以通过依次单击“试用”、“GET 表/todoitem”、“试用此项”和“发送”来验证该数据。有关详细信息，请参阅[在迁移中设定数据种子]。

10. 将移动服务重新发布到 Azure，然后运行客户端应用程序以访问数据，并验证是否可以加载数据且不出错。

13. （可选）在 [Azure 管理门户]中选择你的移动服务，然后单击“配置”>“SQL 数据库”。随后你将导航到移动服务数据库的 SQL 数据库页。

14. （可选）单击“管理”，登录到 SQL 数据库服务器，然后单击“设计”并验证是否已在 Azure 中进行架构更改。


## 在没有初始值设定项的情况下使用 Code First 迁移
在对 .NET 后端项目使用 Code First 迁移之前，应运行数据模型初始值设定项。如果不使用初始值设定项，在尝试应用迁移时可能会出现错误。如果你选择不使用预定义的数据模型初始值设定项之一，请确保在 Migrations\\Configuration.cs 文件中，将迁移配置为使用 EntityTableSqlGenerator 作为 SqlGenerator，如以下示例所示：

    public Configuration()
    {
        AutomaticMigrationsEnabled = false;
        SetSqlGenerator("System.Data.SqlClient", new EntityTableSqlGenerator());
    }

##<a name="seeding"></a>在迁移中设定数据种子

你可以让迁移在执行迁移时向数据库添加种子数据。**Configuration** 类提供了 **Seed** 方法，重写该方法可以插入或更新数据。启用迁移后，Configuration.cs 代码文件将添加到迁移文件夹。以下示例演示了如何重写 [Seed] 方法，以在 **TodoItems** 表中设定数据种子。应在迁移到最新版本后调用 [Seed] 方法。

###设定新表的种子

以下代码将使用新的数据行设定 **TodoItems** 表的种子：

        List<TodoItem> todoItems = new List<TodoItem>
        {
            new TodoItem { Id = "1", Text = "First item", Complete = false },
            new TodoItem { Id = "2", Text = "Second item", Complete = false },
        };

        foreach (TodoItem todoItem in todoItems)
        {
            context.Set<TodoItem>().Add(todoItem);
        }
        base.Seed(context);

###设定表中新列的种子

以下代码只设定 UserId 列的种子：
 		    
        context.TodoItems.AddOrUpdate(
            t => t.UserId,
                new TodoItem { UserId = 1 },
                new TodoItem { UserId = 1 },
                new TodoItem { UserId = 2 }
            );
        base.Seed(context);

此代码将调用 [AddOrUpdate] Helper 扩展方法，以向新的 UserId 列添加种子数据。使用 [AddOrUpdate] 时不会创建重复行。

<!-- Anchors -->
[Migrations]: #migrations
[在迁移中设定数据种子]: #seeding

<!-- Images -->

[0]: ./media/mobile-services-dotnet-backend-how-to-use-code-first-migrations/navagate-to-sql-database.png
[1]: ./media/mobile-services-dotnet-backend-how-to-use-code-first-migrations/manage-sql-database.png
[2]: ./media/mobile-services-dotnet-backend-how-to-use-code-first-migrations/sql-database-drop-tables.png

<!-- URLs -->
[DropCreateDatabaseIfModelChanges]: http://msdn.microsoft.com/zh-cn/library/gg679604(v=vs.113).aspx
[Seed]: http://msdn.microsoft.com/zh-cn/library/hh829453(v=vs.113).aspx
[Azure 管理门户]: https://manage.windowsazure.cn/
[DbContext]: http://msdn.microsoft.com/zh-cn/library/system.data.entity.dbcontext(v=vs.113).aspx
[AddOrUpdate]: http://msdn.microsoft.com/zh-cn/library/system.data.entity.migrations.idbsetextensions.addorupdate(v=vs.103).aspx
[TableController<TEntity>]: https://msdn.microsoft.com/zh-cn/library/azure/dn643359.aspx
[EntityData]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mobile.service.entitydata.aspx
[DbSet<T>]: https://msdn.microsoft.com/zh-cn/library/azure/gg696460.aspx

<!---HONumber=Mooncake_0118_2016-->