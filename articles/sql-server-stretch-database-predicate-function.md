<properties
	pageTitle="使用筛选器谓词来选择要迁移的行（SQL Server Stretch Database）| Azure"
	description="了解如何创建筛选器谓词以选择要迁移的行。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglaslMS"
	manager=""
	editor=""/>

<tags
	ms.service="sql-server-stretch-database"
	ms.date="02/26/2016"
	wacn.date="04/18/2016"/>

# 使用筛选器谓词来选择要迁移的行（SQL Server Stretch Database）

如果在单独的某个表中存储了历史数据，则你可以将SQL Server Stretch Database配置为迁移整个表。另一方面，如果表包含历史数据和当前数据，则你可以指定筛选器谓词来选择要迁移的行。该筛选器谓词必须调用内联表值函数。本主题介绍如何编写内联表值函数用于选择要迁移的行。

在 CTP 3.1 到 RC0 版本中，用于指定谓词的选项在“为数据库启用延伸”向导中不可用。必须将 ALTER TABLE 语句与此选项结合使用才能配置SQL Server Stretch Database。有关详细信息，请参阅 [ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms190273.aspx)。

如果未指定筛选器谓词，则会迁移整个表。

> 如果提供的筛选器谓词性能不佳，则数据迁移的性能也会不佳。SQL Server Stretch Database将使用 CROSS APPLY 运算符对表应用筛选器谓词。

## 内联表值函数的基本要求
SQL Server Stretch Database筛选器函数所需的内联表值函数类似于以下示例。

```tsql
CREATE FUNCTION dbo.fn_stretchpredicate(@column1 datatype1, @column2 datatype2 [, ...n])
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN	SELECT 1 AS is_eligible
		WHERE <predicate>
```
该函数的参数必须是表中列的标识符。

需要使用架构绑定来防止删除或更改筛选器谓词使用的列。

### 返回值
如果该函数返回非空结果，则表示相应的行符合迁移条件；否则 - 即，如果该函数未返回任何行 - 则表示相应的行不符合迁移条件。

### 条件
&lt;谓词&gt; 可以包含一个条件或者使用 AND 逻辑运算符联接的多个条件。

```
<predicate> ::= <condition> [ AND <condition> ] [ ...n ]
```
而每个条件又包含一个基元条件或者使用 OR 逻辑运算符联接的多个基元条件。

```
<condition> ::= <primitive_condition> [ OR <primitive_condition> ] [ ...n ]
```

### 基元条件
基元条件可以执行以下比较之一。

```
<primitive_condition> ::=
{
<function_parameter> <comparison_operator> constant
| <function_parameter> { IS NULL | IS NOT NULL }
| <function_parameter> IN ( constant [ ,...n ] )
}
```

-   将函数参数与常量表达式进行比较。例如，`@column1 < 1000`。

    以下示例将检查 date 列的值是否 `< 1/1/2016`。

    ```tsql
    CREATE FUNCTION dbo.fn_stretchpredicate(@column1 datetime)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 < CONVERT(datetime, '1/1/2016', 101)
    GO

    ALTER TABLE stretch_table_name SET ( REMOTE_DATA_ARCHIVE = ON (
    	FILTER_PREDICATE = dbo.fn_stretchpredicate(date),
    	MIGRATION_STATE = OUTBOUND
    ) )
    ```

-   向函数参数应用 IS NULL 或 IS NOT NULL 运算符。

-   使用 IN 运算符将函数参数与常量值列表进行比较。

    以下示例将检查 shipment\_status 列的值是否为 `IN (N'Completed', N'Returned', N'Cancelled')`。

    ```tsql
    CREATE FUNCTION dbo.fn_stretchpredicate(@column1 nvarchar(15))
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 IN (N'Completed', N'Returned', N'Cancelled')
    GO

    ALTER TABLE table1 SET ( REMOTE_DATA_ARCHIVE = ON (
    	FILTER_PREDICATE = dbo.fn_stretchpredicate(shipment_status),
    	MIGRATION_STATE = OUTBOUND
    ) )
    ```

### 比较运算符
支持以下比较运算符。

`<, <=, >, >=, =, <>, !=, !<, !>`

```
<comparison_operator> ::= { < | <= | > | >= | = | <> | != | !< | !> }
```

### 常量表达式
在筛选器谓词中使用的常量可以是定义函数时可求值的任何确定性表达式。常量表达式可以包含以下内容。

-   文本。例如，`N’abc’, 123`。

-   代数表达式。例如，`123 + 456`。

-   确定性函数。例如，`SQRT(900)`。

-   使用 CAST 或 CONVERT 的确定性转换。例如，`CONVERT(datetime, '1/1/2016', 101)`。

### 其他表达式
如果在将 BETWEEN 和 NOT BETWEEN 运算符替换为等效的 AND 和 OR 表达式后，生成的谓词符合本文所述的规则，则你可以使用这些 BETWEEN 和 NOT BETWEEN 运算符。

不能使用子查询或者类似于 RAND() 或 GETDATE() 的不确定性函数。

## 有效函数的示例

-   以下示例使用 AND 逻辑运算符组合两个基元条件。

    ```tsql
    CREATE FUNCTION dbo.fn_stretchpredicate((@column1 datetime, @column2 nvarchar(15))
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
      WHERE @column1 < N'20150101' AND @column2 IN (N'Completed', N'Returned', N'Cancelled')
    GO

    ALTER TABLE table1 SET ( REMOTE_DATA_ARCHIVE = ON (
    	FILTER_PREDICATE = dbo.fn_stretchpredicate(date, shipment_status),
    	MIGRATION_STATE = OUTBOUND
    ) )
    ```

-   以下示例使用了多个条件，并使用 CONVERT 执行确定性转换。

    ```tsql
    CREATE FUNCTION dbo.fn_stretchpredicate_example1(@column1 datetime, @column2 int, @column3 nvarchar)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
        WHERE @column1 < CONVERT(datetime, '1/1/2015', 101)AND (@column2 < -100 OR @column2 > 100 OR @column2 IS NULL)AND @column3 IN (N'Completed', N'Returned', N'Cancelled')
    GO
    ```

-   以下示例使用数学运算符和函数。

    ```tsql
    CREATE FUNCTION dbo.fn_stretchpredicate_example2(@column1 float)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 < SQRT(400) + 10
    GO
    ```

-   以下示例使用 BETWEEN 和 NOT BETWEEN 运算符。这种用法是有效的，因为在将 BETWEEN 和 NOT BETWEEN 运算符替换为等效的 AND 和 OR 表达式后，生成的谓词符合本文所述的规则。

    ```tsql
    CREATE FUNCTION dbo.fn_stretchpredicate_example3(@column1 int, @column2 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 BETWEEN 0 AND 100
    			AND (@column2 NOT BETWEEN 200 AND 300 OR @column1 = 50)
    GO
    ```
    在将 BETWEEN 和 NOT BETWEEN 运算符替换为等效的 AND 和 OR 表达式后，上面的函数等效于下面的函数。

    ```tsql
    CREATE FUNCTION dbo.fn_stretchpredicate_example4(@column1 int, @column2 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 >= 0 AND @column1 <= 100AND (@column2 < 200 OR @column2 > 300 OR @column1 = 50)
    GO
    ```

## 无效函数的示例

-   以下函数无效，因为它包含不确定性转换。

    ```tsql
    CREATE FUNCTION dbo.fn_example5(@column1 datetime)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 < CONVERT(datetime, '1/1/2016')
    GO
    ```

-   以下函数无效，因为它包含不确定性函数调用。

    ```tsql
    CREATE FUNCTION dbo.fn_example6(@column1 datetime)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 < DATEADD(day, -60, GETDATE())
    GO
    ```

-   以下函数无效，因为它包含子查询。

    ```tsql
    CREATE FUNCTION dbo.fn_example7(@column1 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 IN (SELECT SupplierID FROM Supplier WHERE Status = 'Defunct'))
    GO
    ```

-   以下函数无效，因为使用代数运算符或内置函数的表达式在你定义函数时必须求值为常量。不能在代数表达式或函数调用中包含列引用。

    ```tsql
    CREATE FUNCTION dbo.fn_example8(@column1 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 % 2 =  0
    GO

    CREATE FUNCTION dbo.fn_example9(@column1 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE SQRT(@column1) = 30
    GO
    ```

-   以下函数无效，因为在将 BETWEEN 运算符替换为等效的 AND 表达式后，该函数将违反本文所述的规则。

    ```tsql
    CREATE FUNCTION dbo.fn_example10(@column1 int, @column2 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE (@column1 BETWEEN 1 AND 200 OR @column1 = 300) AND @column2 > 1000
    GO
    ```
    在将 BETWEEN 运算符替换为等效的 AND 表达式后，上面的函数等效于下面的函数。此函数无效，因为基元条件只能使用 OR 逻辑运算符。

    ```tsql
    CREATE FUNCTION dbo.fn_example11(@column1 int, @column2 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE (@column1 >= 1 AND @column1 <= 200 OR @column1 = 300) AND @column2 > 1000
    GO
    ```

## SQL Server Stretch Database如何应用筛选器谓词
SQL Server Stretch Database使用 CROSS APPLY 运算符对表应用筛选器谓词并确定符合条件的行。例如：

```tsql
SELECT * FROM stretch_table_name CROSS APPLY fn_stretchpredicate(column1, column2)
```
如果该函数返回行的非空结果，则表示该行符合迁移条件。

## 将筛选器谓词添加到表中
通过运行 ALTER TABLE 语句并将现有的内联表值函数指定为 FILTER\_PREDICATE 参数值，来向表中添加筛选器谓词。例如：

```tsql
ALTER TABLE stretch_table_name SET ( REMOTE_DATA_ARCHIVE = ON (
	FILTER_PREDICATE = dbo.fn_stretchpredicate(column1, column2),
	MIGRATION_STATE = <desired_migration_state>
) )
```
将函数作为谓词绑定到表后，将发生以下情况。

-   下一次执行数据迁移时，只会迁移该函数对其返回了非空值的行。

-   该函数使用的列已绑定到架构。只要表使用函数作为筛选器谓词，你就不能更改这些列。

## 从表中删除筛选器谓词
若要迁移整个表而不是选定的行，可以通过将现有 FILTER\_PREDICATE 设置为 null 来删除它。例如：

```tsql
ALTER TABLE stretch_table_name SET ( REMOTE_DATA_ARCHIVE = ON (
	FILTER_PREDICATE = NULL,
	MIGRATION_STATE = <desired_migration_state>
) )
```
删除筛选器谓词后，表中的所有行都符合迁移的条件。

## 替换现有的筛选器谓词
可以通过再次运行 ALTER TABLE 语句并指定 FILTER\_PREDICATE 参数的新值，来替换以前指定的筛选器谓词。例如：

```tsql
ALTER TABLE stretch_table_name SET ( REMOTE_DATA_ARCHIVE = ON (
	FILTER_PREDICATE = dbo.fn_stretchpredicate2(column1, column2),
	MIGRATION_STATE = <desired_migration_state>
```
新的内联表值函数具有以下要求。

-   新函数的限制强度必须低于以前的函数。

-   旧函数中的所有运算符必须存在于新函数中。

-   新函数不能包含旧函数中不存在的运算符。

-   运算符参数的顺序不能更改。

-   只能更改 `<, <=, >, >=` 比较表达式中的常量值，但更改的方式必须使谓词的限制性更低。

### 有效替换的示例
假设以下函数是当前的筛选器谓词。

```tsql
CREATE FUNCTION dbo.fn_stretchpredicate_old (@column1 datetime, @column2 int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN	SELECT 1 AS is_eligible
		WHERE @column1 < CONVERT(datetime, '1/1/2016', 101)
			AND (@column2 < -100 OR @column2 > 100)
GO
```
以下函数是有效的替换，因为新日期常量（指定一个更迟的截止日期）会使该谓词的限制强度更低。

```tsql
CREATE FUNCTION dbo.fn_stretchpredicate_new (@column1 datetime, @column2 int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN	SELECT 1 AS is_eligible
		WHERE @column1 < CONVERT(datetime, '2/1/2016', 101)
			AND (@column2 < -50 OR @column2 > 50)
GO
```

### 无效替换的示例
以下函数不是有效的替换，因为新日期常量（指定一个更早的截止日期）不能使该谓词的限制强度更低。

```tsql
CREATE FUNCTION dbo.fn_notvalidreplacement_1 (@column1 datetime, @column2 int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN	SELECT 1 AS is_eligible
		WHERE @column1 < CONVERT(datetime, '1/1/2015', 101)
			AND (@column2 < -100 OR @column2 > 100)
GO
```
以下函数不是有效的替换，因为删除了某个比较运算符。

```tsql
CREATE FUNCTION dbo.fn_notvalidreplacement_2 (@column1 datetime, @column2 int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN	SELECT 1 AS is_eligible
		WHERE @column1 < CONVERT(datetime, '1/1/2016', 101)
			AND (@column2 < -50)
GO
```
以下函数不是有效的替换，因为使用 AND 逻辑运算符添加了新的条件。

```tsql
CREATE FUNCTION dbo.fn_notvalidreplacement_3 (@column1 datetime, @column2 int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN	SELECT 1 AS is_eligible
		WHERE @column1 < CONVERT(datetime, '1/1/2016', 101)
			AND (@column2 < -100 OR @column2 > 100)
			AND (@column2 <> 0)
GO
```

## 删除筛选器谓词
只要表使用函数作为筛选器谓词，你就不能删除内联表值函数。

## 检查应用于表的筛选器谓词
若要检查应用于表的筛选器谓词，请打开目录视图 **sys.remote\_data\_archive\_tables** 并检查 **filter\_predicate** 列的值。如果值为 null，则整个表符合存档条件。有关详细信息，请参阅 [sys.remote\_data\_archive\_tables (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/dn935003.aspx)。

## 另请参阅
[ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms190273.aspx)

<!---HONumber=Mooncake_0411_2016-->