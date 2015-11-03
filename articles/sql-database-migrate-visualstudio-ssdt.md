<properties 
   pageTitle="使用 Visual Studio 和 SSDT 进行迁移" 
   description="Microsoft Azure SQL 数据库, 数据库迁移, 导入数据库, 导出数据库, 迁移向导" 
   services="sql-database" 
   documentationCenter="" 
   authors="carlrabeler" 
   manager="jeffreyg" 
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="08/24/2015"
   wacn.date="11/02/2015"/>

#就地更新数据库，然后部署到 Azure SQL 数据库

![替换文字](./media/sql-database-migrate-visualstudio-ssdt/01VSSSDTDiagram.png)

如果将本地数据库迁移到 Azure SQL 数据库 V12 时需要更改架构（因为该数据库使用 Azure SQL 数据库不支持的 SQL Server 功能），或者要测试本地数据库中是否存在不支持的功能，请使用此选项。

请使用包含 Visual Studio 2013 Update 4 或更高版本的[最新 SQL Server Data Tools for Visual Studio](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)。

使用此选项：

 - 将首先使用用于 Visual Studio 的 SQL Server Data Tools（简称“SSDT”）从源数据库创建数据库项目。 
 - 然后将该项目目标平台设为 Azure SQL 数据库 V12，并生成项目以确定所有兼容性问题。 
 - 成功生成项目后，将架构发布回源数据库的副本（不覆盖源数据库）。
 - 然后使用 SSDT 中的数据比较功能将源数据库与新创建的 Azure SQL 兼容数据库进行比较，然后使用源数据库中的数据更新新数据库。 
 - 然后使用 SSMS 直接或通过导出方式将已更新的数据库部署到 Azure，然后导入 BACPAC 文件。
 
>[AZURE.NOTE] 注意：如果需要仅有架构的部署，可将更新的架构从 Visual Studio 直接发布到 Azure SQL 数据库。

## 迁移步骤

1.	在 Visual Studio 中打开“SQL Server 对象资源管理器”。使用“添加 SQL Server ”连接到包含被迁移数据库的 SQL Server 实例。在资源管理器中找到数据库，右键单击它并选择“创建新项目...” 

	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/02MigrateSSDT.png)

2.	将导入设置配置为“仅导入应用程序范围对象”。取消选中导入以下项的选项：引用的登录名、权限和数据库设置。

	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/03MigrateSSDT.png)

3.	单击“启动”以导入数据库并创建项目，项目将包含数据库中每个对象的 T-SQL 脚本文件。这些脚本文件嵌入到项目中的文件夹内。

	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/04MigrateSSDT.png)

4.	在 Visual Studio 解决方案资源管理器中，右键单击数据库项目并选择“属性”。此操作将打开“项目设置”页，在该页上应将“目标平台”配置为“Microsoft Azure SQL 数据库 V12”。

	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/05MigrateSSDT.png)

5.	右键单击项目并选择“生成”以生成项目。

	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/06MigrateSSDT.png)

6.	**错误列表**将显示每个不兼容项。在此示例中，用户名 NT AUTHORITY\\NETWORK SERVICE 是不兼容的。由于它不兼容，你可以将它注释掉或删除（并解决从数据库解决方案中删除此登录名和角色的影响）。 

	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/07MigrateSSDT.png)
  
7.	双击第一个脚本以在查询窗口中打开该脚本，注释掉该脚本，然后执行该脚本。 
	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/08MigrateSSDT.png)

8.	对包含不兼容项的每个脚本重复执行此过程，直到不再存在错误。
	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/09MigrateSSDT.png)
 
9.	当数据库无错误后，右键单击项目并选择“发布”以生成数据库并将其发布到源数据库副本（强烈建议至少在开始时使用副本）。 
 - 在发布前，根据源 SQL Server 版本（早于 SQL Server 2014），可能需要重置项目的目标平台才能进行部署。 
 - 如果迁移的是较早的 SQL Server 数据库，切勿向项目中引入任何不受源 SQL Server 支持的功能，除非先将数据库迁移到较新版本的 SQL Server。 

	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/10MigrateSSDT.png)

	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/11MigrateSSDT.png)

10.	在 SQL Server 对象资源管理器中，右键单击你的源数据库，然后单击“数据比较”，将项目与原始数据库进行比较以了解向导进行了哪些更改。选择 Azure SQL V12 版本的数据库，然后单击“完成”。

	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/12MigrateSSDT.png)

	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/13MigrateSSDT.png)

12.	查看检测到的差异，然后单击“更新目标”，以将源数据库中的数据迁移到 Azure SQL V12 数据库。

	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/14MigrateSSDT.png)

14.	使用 SSMS 将与 Azure SQL V12 兼容的数据库架构和数据部署到 Azure SQL 数据库。请参阅[使用 SSMS 迁移兼容数据库](/documentation/articles/sql-database-migrate-ssms)。

<!---HONumber=76-->