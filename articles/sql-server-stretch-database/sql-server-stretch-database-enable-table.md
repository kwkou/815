<properties
	pageTitle="为表启用 Stretch Database | Azure"
	description="了解如何为延伸数据库配置表。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglaslMS"
	manager=""
	editor=""/>

<tags
	ms.service="sql-server-stretch-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/05/2016"
	wacn.date="09/05/2016"
	ms.author="douglasl"/>

# 为表启用延伸数据库

若要为延伸数据库配置某个表，请在 SQL Server Management Studio 中选择该表对应的“任务 | 延伸 | 启用”，打开“为表启用延伸”向导。你也可以使用 Transact-SQL 对现有表启用 Stretch Database，或使用启用的 Stretch Database 创建一个新表。

-   如果在单独的某个表中存储了冷数据，你可以迁移整个表。

-   如果表同时包含热数据和冷数据，你可以指定筛选器函数来选择要迁移的行。

**先决条件**。如果你针对某个表选择了“延伸 | 启用”但尚未为数据库启用延伸数据库，该向导将先为延伸数据库配置数据库。请遵循[通过运行“为数据库启用 Stretch 向导”开始操作](/documentation/articles/sql-server-stretch-database-wizard/)中的步骤，而不要遵循本主题中的步骤。

**权限**。对数据库或表启用延伸数据库需要有 db\_owner 权限。对某个表启用延伸数据库还要求对该表拥有 ALTER 权限。

 >   [AZURE.NOTE] 若要在以后禁用 Stretch Database，请记住，针对表或数据库禁用 Stretch Database 不会删除远程对象。若要删除远程表或远程数据库，必须使用 Azure 管理门户。远程对象在手动删除之前，会持续产生 Azure 费用。
 
## <a name="EnableWizardTable"></a>使用向导来对表启用 Stretch Database
**启动向导**

1.  在 SQL Server Management Studio 的对象资源管理器中，选择你要为其启用延伸的表。

2.  单击右键并选择“Stretch”，然后选择“启用”以启动向导。

**介绍**

查看向导的用途和先决条件。

**选择数据库表**

确认你要启用的表已显示并已选中。

你可以迁移整个表，也可以在向导中指定一个简单的筛选器函数。如果想使用不同类型的筛选器函数来选择要迁移的行，请执行下列操作之一。

-   退出向导并运行 ALTER TABLE 语句，以便为表启用延伸，并指定筛选器函数。

-   退出向导后，运行 ALTER TABLE 语句，以便指定筛选器函数。有关所需步骤，请参阅[运行向导后添加筛选器函数](/documentation/articles/sql-server-stretch-database-predicate-function/#addafterwiz)。

本主题稍后将介绍 ALTER TABLE 语法。

**摘要**

查看在向导中输入的值和选择的选项。然后选择“完成”以启用延伸。

**结果**

查看结果。

## <a name="EnableTSQLTable"></a>使用 Transact-SQL 对表启用 Stretch Database
你也可以使用 Transact-SQL 为现有表启用 Stretch Database，或使用启用的 Stretch Database 创建一个新表。

### 选项
在运行 CREATE TABLE 或 ALTER TABLE 来对表启用 Stretch Database 时，请使用以下选项。

-   （可选）如果表同时包含热数据和冷数据，请使用 `FILTER_PREDICATE = <function>` 子句指定一个函数来选择要迁移的行。该谓词必须调用内联表值函数。有关详细信息，请参阅[使用筛选器函数选择要迁移的行](/documentation/articles/sql-server-stretch-database-predicate-function/)。如果未指定筛选器函数，则会迁移整个表。

    >   [AZURE.NOTE] 如果提供的筛选器函数性能不佳，则数据迁移的性能也会不佳。Stretch Database 使用 CROSS APPLY 运算符对表应用筛选器函数。

-   指定 `MIGRATION_STATE = OUTBOUND` 立即开始数据迁移，或指定 `MIGRATION_STATE = PAUSED` 推迟数据迁移的开始时间。

### 为现有表启用 Stretch Database
若要为 Stretch Database 配置现有表，请运行 ALTER TABLE 命令。

以下示例将迁移整个表并立即开始数据迁移。

	USE <Stretch-enabled database name>;
	GO
	ALTER TABLE <table name>  
	    SET ( REMOTE_DATA_ARCHIVE = ON ( MIGRATION_STATE = OUTBOUND ) ) ;  
	GO
以下示例只会迁移 `dbo.fn_stretchpredicate` 内联表值函数识别的行，并推迟数据迁移。有关筛选器函数的详细信息，请参阅[使用筛选器函数选择要迁移的行](/documentation/articles/sql-server-stretch-database-predicate-function/)。

	USE <Stretch-enabled database name>;
	GO
	ALTER TABLE <table name>  
	    SET ( REMOTE_DATA_ARCHIVE = ON (  
	        FILTER_PREDICATE = dbo.fn_stretchpredicate(),  
	        MIGRATION_STATE = PAUSED ) ) ;  
	 GO

有关详细信息，请参阅 [ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms190273.aspx)。

### 使用启用的 Stretch Database 创建一个新表
若要使用启用的 Stretch Database 创建一个新表，请运行 CREATE TABLE 命令。

以下示例将迁移整个表并立即开始数据迁移。


	USE <Stretch-enabled database name>;
	GO
	CREATE TABLE <table name>
	    ( ... )  
	    WITH ( REMOTE_DATA_ARCHIVE = ON ( MIGRATION_STATE = OUTBOUND ) ) ;  
	GO

以下示例只会迁移 `dbo.fn_stretchpredicate` 内联表值函数识别的行，并推迟数据迁移。有关筛选器函数的详细信息，请参阅[使用筛选器函数选择要迁移的行](/documentation/articles/sql-server-stretch-database-predicate-function/)。

	USE <Stretch-enabled database name>;
	GO
	CREATE TABLE <table name>
	    ( ... )  
	    WITH ( REMOTE_DATA_ARCHIVE = ON (  
	        FILTER_PREDICATE = dbo.fn_stretchpredicate(),  
	        MIGRATION_STATE = PAUSED ) ) ;  
	GO  

有关详细信息，请参阅 [CREATE TABLE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms174979.aspx)。


## 另请参阅

[ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms190273.aspx)

[CREATE TABLE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms174979.aspx)

<!---HONumber=Mooncake_0829_2016-->