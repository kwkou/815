<properties pageTitle="How to use Code First Migrations .NET backend (Mobile Services)" metaKeywords="" description="" metaCanonical="" services="" documentationCenter="" title="Considerations for supporting multiple clients from a single mobile service" authors="glenga" solutions="" writer="glenga" manager="dwrede" editor="" />

# 如何对 .NET 后端移动服务进行数据模型更改

在 .NET 后端移动服务项目中，默认的 Entity Framework Code First 数据库初始值设定项从 [DropCreateDatabaseIfModelChanges][] 类派生。每当此初始值设定项检测到 [DbContext][] 公开的数据模型更改时，都会通知 Entity Framework 删除然后重新创建数据库。在本地开发移动服务项目期间，你应该持续使用此初始值设定项，而 .NET 后端教程也假设你使用的是此初始值设定项。但是，如果你要做出数据模型更改并在数据库中维护现有数据，则必须使用 Code First 迁移。若要将数据模型更改发布到 Azure，使用 Code First 迁移也是一个不错的解决方案，因为 SQL Database 不可删除。

本主题说明如何使用 Code First 迁移对现有的 SQL Database 进行数据模型更改且不丢失现有数据。此过程假设你已将移动服务项目发布到 Azure，数据库中已有数据，并且远程和本地数据模型仍然保持同步。

> [WACOM.NOTE]在发布到 Azure 之前，我们建议你尽量在本地计算机上完成数据模型开发。如果你已将 .NET 后端移动服务项目发布到 Azure，而你的 SQL Database 表架构与项目的当前数据模型不匹配，则你必须删除表，或者手动使它们同步，然后再尝试使用 Code First 迁移进行发布。

在本地计算机上开发 .NET 后端移动服务项目时，处理数据模型更改的最简单方法是持续使用默认的初始值设定项，每当检测到数据模型更改时，该初始值设定项将会删除然后重新创建数据库。将项目重新发布到 Azure 时不能使用这种方法。在此情况下，由于运行时无权删除 Azure 中的 SQL Database（这是一个合理的安全机制），因此初始值设定项将会失败。

> [WACOM.NOTE]针对实时 Azure 服务开发和测试移动服务项目时，始终应该使用专门用于测试的移动服务实例。切勿针对当前已投入生产或者正由客户端应用程序使用的移动服务开发或测试项目。

## 删除 SQL Database 中的表

在 Azure 中针对 SQL Database 执行迁移之前，应该先手动删除数据库架构中由移动服务使用的所有现有表。执行以下步骤可以从 SQL Database 中删除现有的表。如果数据库架构已经与当前数据模型同步，则你可以跳过此过程，直接开始[迁移][]。

1.  登录到 [Azure 管理门户][]，选择你的移动服务，单击“配置”选项卡，然后单击“SQL Database”链接 。

    ![][]

    随后你将转到移动服务使用的数据库的门户页。

2.  单击“管理”按钮并登录到 SQL Database 服务器 。

    ![][1]

3.  在 SQL Database 管理器中，依次单击“设计”和“表”，选择移动服务架构中的某个表，单击“删除表”，然后单击“确定”进行确认 。

    ![][2]

4.  针对移动服务架构中的每个表重复上述步骤。

    删除现有的表后，可以在 SQL Database 上初始化 Code First 迁移。不属于移动服务架构的表不会影响你的移动服务，因此不应将其删除。

<a name="migrations"></a>
## 启用 Code First 迁移

Code First 迁移使用快照方法来生成代码，执行这些代码会对数据库进行架构更改。借助迁移，你可对数据模型进行增量更改并在数据库中维护现有数据。以下步骤将启用迁移，并应用项目、本地数据库和 Azure 中的数据模型更改。

1.  在 Visual Studio 的解决方案资源管理器中，右键单击移动服务项目，然后单击“设为启动项目” 。

2.  在“工具” 菜单中，展开“NuGet 包管理器” ，然后单击“程序包管理器控制台” 。

    此时会显示程序包管理器控制台，你可以使用它来管理 Code First 迁移。

3.  在程序包管理器控制台中运行以下命令：

        PM> Enable-Migrations

    这将为你的项目启用 Code First 迁移。

4.  在控制台中运行以下命令：

        PM> Add-Migration Initial

    这将创建一个名为 *Initial* 的新迁移。迁移代码存储在迁移项目文件夹中。

5.  展开 App\_Start 文件夹，打开 WebApiConfig.cs 项目文件并添加以下 "using" 语句：

        using System.Data.Entity.Migrations;
        using todolistService.Migrations;

    在上述代码中，必须将 *todolistService* 字符串替换为项目的命名空间，对于下载的快速入门项目，该命名空间为 *mobile\_service\_name*Service。

6.  在这同一个代码文件中，注释掉对 "Database.SetInitializer" 方法的调用，并在该调用的后面添加以下代码：

        var migrator = new DbMigrator(new Configuration());
        migrator.Update();

    这样就会禁用用于删除然后重新创建数据库的默认 Code First 数据库初始值设定项，并将其替换为应用最新迁移的显式请求。此时，除非为数据创建了迁移，否则，在访问数据时，进行任何数据模型更改都会导致 InvalidOperationException。另外，你的服务必须使用 Code First 迁移将数据模型更改迁移到数据库。

7.  按 F5 在本地计算机上启动移动服务项目。

    此时，数据库已与数据模型同步。如果你提供了种子数据，可以通过依次单击“试用”、“GET 表/todoitem”、“试用此项”和“发送”来验证该数据 。有关详细信息，请参阅[在迁移中设定数据种子][]。

8.  现在，对数据模型进行更改（例如，向 TodoItem 类型添加一个新的 UserId 属性），重新生成项目，然后在程序包管理器中运行以下命令：

        PM> Add-Migration NewUserId

    这将创建一个名为 *NewUserId* 的新迁移。迁移文件夹中添加了一个用于实施此更改的新代码文件

9.  再次按 F5 以在本地计算机上重新启动移动服务项目。

    迁移已应用到数据库，并且数据库已重新与数据模型同步。如果你提供了种子数据，可以通过依次单击“试用”、“GET 表/todoitem”、“试用此项”和“发送”来验证该数据 。有关详细信息，请参阅[在迁移中设定数据种子][]。

10. 将移动服务重新发布到 Azure，然后运行客户端应用程序以访问数据，并验证是否可以加载数据且不出错。

11. （可选）在 [Azure 管理门户][]中选择你的移动服务，单击“配置”选项卡，然后单击“SQL Database”链接 。

    ![][]

    随后你将导航到移动服务数据库的 SQL Database 页。

12. （可选）单击“管理”，登录到 SQL Database 服务器，然后单击“设计”并验证是否已在 Azure 中进行架构更改 。

    ![][1]

<a name="seeding"></a>
## 在迁移中设定数据种子

你可以让迁移在执行迁移时向数据库添加种子数据。Configuration 类提供了 Seed 方法，重写该方法可以插入或更新数据。启用迁移后，Configuration.cs 代码文件将添加到迁移文件夹。以下示例演示了如何重写 [Seed][] 方法，以在 "TodoItems" 表中设定数据种子。应在迁移到最新版本后调用 [Seed][] 方法。

### 设定新表的种子

以下代码将使用新的数据行设定 "TodoItems" 表的种子：

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

### 设定表中新列的种子

以下代码只设定 UserId 列的种子：

        context.TodoItems.AddOrUpdate(
    t => t.UserId,
    new TodoItem { UserId = 1 },
    new TodoItem { UserId = 1 },
    new TodoItem { UserId = 2 }
            );
    base.Seed(context);

此代码将调用 [AddOrUpdate][] Helper 扩展方法，以向新的 UserId 列添加种子数据。使用 [AddOrUpdate][] 时不会创建重复行。

  [DropCreateDatabaseIfModelChanges]: http://msdn.microsoft.com/zh-cn/library/gg679604(v=vs.113).aspx
  [DbContext]: http://msdn.microsoft.com/zh-cn/library/system.data.entity.dbcontext(v=vs.113).aspx
  [迁移]: #migrations
  [Azure 管理门户]: https://manage.windowsazure.cn/
  []: ./media/mobile-services-dotnet-backend-how-to-use-code-first-migrations/navagate-to-sql-database.png
  [1]: ./media/mobile-services-dotnet-backend-how-to-use-code-first-migrations/manage-sql-database.png
  [2]: ./media/mobile-services-dotnet-backend-how-to-use-code-first-migrations/sql-database-drop-tables.png
  [在迁移中设定数据种子]: #seeding
  [Seed]: http://msdn.microsoft.com/zh-cn/library/hh829453(v=vs.113).aspx
  [AddOrUpdate]: http://msdn.microsoft.com/zh-cn/library/system.data.entity.migrations.idbsetextensions.addorupdate(v=vs.103).aspx
