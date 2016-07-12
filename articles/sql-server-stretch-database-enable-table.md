<properties
	pageTitle="为表启用延伸数据库 | Azure"
	description="了解如何为延伸数据库配置表。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglaslMS"
	manager=""
	editor=""/>

<tags
	ms.service="sql-server-stretch-database"
	ms.date="02/26/2016"
	wacn.date="04/05/2016"/>

# 为表启用SQL Server Stretch Database

若要为SQL Server Stretch Database配置某个表，请在 SQL Server Management Studio 中选择该表对应的“任务 | 延伸 | 启用”，打开“为表启用延伸”向导。你也可以使用 Transact-SQL 对现有表启用 Stretch Database，或使用启用的 Stretch Database 创建一个新表。

-   如果在单独的某个表中存储了历史数据，你可以迁移整个表。

-   如果表包含历史数据和当前数据，你可以指定筛选器谓词来选择要迁移的行。在 CTP 3.1 到 RC1 版本中，用于指定筛选器谓词的选项在“为数据库启用延伸”向导中不可用。必须将 CREATE TABLE 或 ALTER TABLE 语句与此选项结合使用才能为SQL Server Stretch Database配置表。

**先决条件**。如果你针对某个表选择了“延伸 | 启用”但尚未为数据库启用SQL Server Stretch Database，该向导将先为SQL Server Stretch Database配置数据库。请遵循[为延伸向导启用数据库](/documentation/articles/sql-server-stretch-database-wizard/)中的步骤，而不要遵循本主题中的步骤。

**权限**。对数据库或表启用SQL Server Stretch Database需要有 db\_owner 权限。对某个表启用SQL Server Stretch Database还要求对该表拥有 ALTER 权限。

## <a name="EnableWizardTable"></a>使用向导来对表启用SQL Server Stretch Database
**启动向导**

1.  在 SQL Server Management Studio 的对象资源管理器中，选择你要为其启用延伸的表。

2.  单击右键并选择“延伸”，然后选择“启用”以启动向导。

**介绍**

查看向导的用途和先决条件。

**选择数据库表**

确认你要启用的表已显示并已选中。

在 CTP 3.1 到 RC1 版本中，你只能使用向导迁移整个表。如果你要指定一个谓词以便在包含历史数据和当前数据的表中选择要迁移的行，请在退出向导之后运行 ALTER TABLE 语句指定谓词，或者如本主题稍后所述退出向导，然后运行 ALTER TABLE 语句。

**摘要**

查看在向导中输入的值和选择的选项。然后选择“完成”以启用延伸。

**结果**

查看结果。

## <a name="EnableTSQLTable"></a>使用 Transact-SQL 对表启用SQL Server Stretch Database
你也可以使用 Transact-SQL 为现有表启用 Stretch Database，或使用启用的 Stretch Database 创建一个新表。

### 常用选项
在运行 CREATE TABLE 或 ALTER TABLE 来对表启用 Stretch Database 时，请使用以下选项。

1.  （可选）如果表包含历史数据和当前数据，请使用 `FILTER_PREDICATE = <predicate>` 子句指定一个谓词来选择要迁移的行。该谓词必须调用内联表值函数。有关详细信息，请参阅[编写内联表值函数以选择行（SQL Server Stretch Database）](/documentation/articles/sql-server-stretch-database-predicate-function/)。如果未指定筛选器谓词，则会迁移整个表。

    > 如果提供的筛选器谓词性能不佳，则数据迁移的性能也会不佳。SQL Server Stretch Database将使用 CROSS APPLY 运算符对表应用筛选器谓词。

    在 CTP 3.1 到 RC1 版本中，此选项在“为数据库启用延伸”向导中不可用。必须将 CREATE TABLE 或 ALTER TABLE 语句与此选项结合使用才能为SQL Server Stretch Database配置表。有关详细信息，请参阅 [ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms190273.aspx)。

-   指定 `MIGRATION_STATE = OUTBOUND` 立即开始数据迁移，或指定 `MIGRATION_STATE = PAUSED` 推迟数据迁移的开始时间。

### 为现有表启用 Stretch Database
若要为 Stretch Database 配置现有表，请运行 ALTER TABLE 命令。

以下示例将迁移整个表并立即开始数据迁移。

```tsql
ALTER TABLE <table name>
    SET ( REMOTE_DATA_ARCHIVE = ON ( MIGRATION_STATE = OUTBOUND ) ) ;
```
以下示例只会迁移 `dbo.fn_stretchpredicate` 内联表值函数识别的行，并推迟数据迁移。有关筛选器谓词的详细信息，请参阅[编写内联表值函数以选择行（SQL Server Stretch Database）](/documentation/articles/sql-server-stretch-database-predicate-function/)。

```tsql
ALTER TABLE <table name>
    SET ( REMOTE_DATA_ARCHIVE = ON (
        FILTER_PREDICATE = dbo.fn_stretchpredicate(date),
        MIGRATION_STATE = PAUSED ) )
```

有关详细信息，请参阅 [ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms190273.aspx)。

### 使用启用的 Stretch Database 创建一个新表
若要使用启用的 Stretch Database 创建一个新表，请运行 CREATE TABLE 命令。

以下示例将迁移整个表并立即开始数据迁移。

```tsql
CREATE TABLE <table name> ...
    WITH ( REMOTE_DATA_ARCHIVE = ON ( MIGRATION_STATE = OUTBOUND ) ) ;
```
以下示例只会迁移 `dbo.fn_stretchpredicate` 内联表值函数识别的行，并推迟数据迁移。有关筛选器谓词的详细信息，请参阅[编写内联表值函数以选择行（延伸数据库）](/documentation/articles/sql-server-stretch-database-predicate-function/)。

```tsql
CREATE TABLE <table name> ...
    WITH ( REMOTE_DATA_ARCHIVE = ON (
        FILTER_PREDICATE = dbo.fn_stretchpredicate(date),
        MIGRATION_STATE = PAUSED ) );
```

有关详细信息，请参阅 [CREATE TABLE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms174979.aspx)。


## 另请参阅

[ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms190273.aspx)

[CREATE TABLE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms174979.aspx)

<!---HONumber=Mooncake_0328_2016-->