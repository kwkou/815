<properties
    pageTitle="选择要针对 Stretch Database 进行迁移的行 - Azure | Azure"
    description="了解如何使用筛选器函数来选择要迁移的行。"
    services="sql-server-stretch-database"
    documentationcenter=""
    author="douglaslMS"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="f5ef79d9-68ef-4394-a057-d7aac5706b72"
    ms.service="sql-server-stretch-database"
    ms.workload="data-management"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="06/28/2016"
    wacn.date="03/24/2017"
    ms.author="douglasl" />  

# 使用筛选器函数来选择要迁移的行 (Stretch Database)

如果在单独的某个表中存储了冷数据，则可以将 Stretch Database 配置为迁移整个表。另一方面，如果表同时包含热数据和冷数据，则可以指定筛选器函数来选择要迁移的行。筛选器谓词是一个内联表值函数。本主题介绍如何编写内联表值函数用于选择要迁移的行。

> [AZURE.NOTE]
> 如果提供的筛选器函数性能不佳，数据迁移的性能也会不佳。Stretch Database 使用 CROSS APPLY 运算符对表应用筛选器函数。
>
>

如果未指定筛选器函数，则会迁移整个表。

运行“启用数据库延伸”向导时，可以迁移整个表，也可以在向导中指定简单的筛选器函数。如果想使用不同类型的筛选器函数来选择要迁移的行，请执行下列操作之一。

* 退出向导并运行 ALTER TABLE 语句，对表启用延伸，并指定筛选器函数。

* 退出向导后，运行 ALTER TABLE 语句，指定筛选器函数。

本主题稍后将介绍用于添加函数的 ALTER TABLE 语法。

## 筛选器函数的基本要求
Stretch Database 筛选器谓词所需的内联表值函数类似于以下示例。

    CREATE FUNCTION dbo.fn_stretchpredicate(@column1 datatype1, @column2 datatype2 [, ...n])
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE <predicate>

该函数的参数必须是表中列的标识符。

需要使用架构绑定来防止删除或更改筛选器函数所使用的列。

### 返回值
如果该函数返回非空结果，表示该行符合迁移条件。否则（即，如果该函数未返回结果），表示该行不符合迁移条件。

### 条件
&lt;*谓词*&gt; 可以包含一个条件或使用 AND 逻辑运算符联接的多个条件。


    <predicate> ::= <condition> [ AND <condition> ] [ ...n ]

而每个条件又包含一个基元条件或者使用 OR 逻辑运算符联接的多个基元条件。


    <condition> ::= <primitive_condition> [ OR <primitive_condition> ] [ ...n ]


### 基元条件
基元条件可以执行以下比较之一。

    <primitive_condition> ::=
    {
    <function_parameter> <comparison_operator> constant
    | <function_parameter> { IS NULL | IS NOT NULL }
    | <function_parameter> IN ( constant [ ,...n ] )
    }


* 将函数参数与常量表达式进行比较。例如，`@column1 < 1000`。

    以下示例将检查 *date* 列的值是否为 &lt; 2016/1/1。

    
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
    

* 向函数参数应用 IS NULL 或 IS NOT NULL 运算符。

* 使用 IN 运算符将函数参数与常量值列表进行比较。

    以下示例将检查 *shipment\_status* 列的值是否为 `IN (N'Completed', N'Returned', N'Cancelled')`。

    
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
    

### 比较运算符
支持以下比较运算符。

`<, <=, >, >=, =, <>, !=, !<, !>`


    <comparison_operator> ::= { < | <= | > | >= | = | <> | != | !< | !> }


### 常量表达式
在筛选器函数中使用的常量可以是定义函数时可求值的任何确定性表达式。常量表达式可以包含以下内容。

* 文本。例如，`N'abc', 123`。

* 代数表达式。例如，`123 + 456`。

* 确定性函数。例如，`SQRT(900)`。

* 使用 CAST 或 CONVERT 的确定性转换。例如，`CONVERT(datetime, '1/1/2016', 101)`。

### 其他表达式
如果在将 BETWEEN 和 NOT BETWEEN 运算符替换为等效的 AND 和 OR 表达式后，生成的函数符合本文所述的规则，则可以使用这些 BETWEEN 和 NOT BETWEEN 运算符。

不能使用子查询或者类似于 RAND() 或 GETDATE() 的不确定性函数。

## 向表中添加筛选器函数
通过运行 **ALTER TABLE** 语句并将现有的内联表值函数指定为 **FILTER\_PREDICATE** 参数的值，来向表中添加筛选器函数。例如：


    ALTER TABLE stretch_table_name SET ( REMOTE_DATA_ARCHIVE = ON (
    	FILTER_PREDICATE = dbo.fn_stretchpredicate(column1, column2),
    	MIGRATION_STATE = <desired_migration_state>
    ) )

将函数作为谓词绑定到表后，将发生以下情况。

* 下一次执行数据迁移时，只会迁移该函数对其返回了非空值的行。

* 该函数使用的列已绑定到架构。只要表使用函数作为筛选器谓词，你就不能更改这些列。

只要表使用函数作为筛选器谓词，就不能删除内联表值函数。

> [AZURE.NOTE]
> 若要提高筛选器函数的性能，请在该函数使用的列上创建索引。
>
>

### 将列名传递给筛选器函数
向表分配筛选器函数时，使用一部分名称指定传递给筛选器函数的列名。如果传递列名时指定三部分名称，则针对已启用延伸的表进行的后续查询将失败。

例如，如果指定如以下示例中所示的三部分列名，该语句将成功运行，但对该表的后续查询将失败。


    ALTER TABLE SensorTelemetry
      SET ( REMOTE_DATA_ARCHIVE = ON (
        FILTER_PREDICATE=dbo.fn_stretchpredicate(dbo.SensorTelemetry.ScanDate),
        MIGRATION_STATE = OUTBOUND )
      )

请改为使用一部分列名指定筛选器函数，如以下示例所示。

    ALTER TABLE SensorTelemetry
      SET ( REMOTE_DATA_ARCHIVE = ON  (
        FILTER_PREDICATE=dbo.fn_stretchpredicate(ScanDate),
        MIGRATION_STATE = OUTBOUND )
      )


## <a name="addafterwiz"></a>在运行向导后添加筛选器函数  

如果要使用在“启用数据库延伸”向导中无法创建的函数，可以在退出该向导后运行 ALTER TABLE 语句来指定函数。但是，在应用函数之前，必须停止已在进行的数据迁移并取回已迁移的数据。（有关必须进行此操作的原因的详细信息，请参阅[替换现有的筛选器函数](#replacePredicate)）。

1. 反向迁移，并取回已迁移的数据。启动此操作后将无法取消此操作。另外，Azure 上会产生出站数据传输（传出）的费用。有关详细信息，请参阅 [Azure 如何定价](/pricing/details/data-transfer/)。

      
        ALTER TABLE <table name>  
             SET ( REMOTE_DATA_ARCHIVE ( MIGRATION_STATE = INBOUND ) ) ;   
      

2. 等待迁移完成。可以在 SQL Server Management Studio 的 **延伸数据库监视器**中查看状态，也可以查询 **sys.dm\_db\_rda\_migration\_status** 视图。有关详细信息，请参阅[数据迁移的监视和故障排除](/documentation/articles/sql-server-stretch-database-monitor/) 或 [sys.dm\_db\_rda\_migration\_status](https://msdn.microsoft.com/zh-cn/library/dn935017.aspx)。

3. 创建要应用于表的筛选器函数。

4. 将该函数添加到表，并重新开始将数据迁移到 Azure。

      
        ALTER TABLE <table name>  
            SET ( REMOTE_DATA_ARCHIVE  
                (           
                    FILTER_PREDICATE = <predicate>,  
                    MIGRATION_STATE = OUTBOUND  
                )  
            );   
      

## 按日期筛选行
下面的示例将迁移 **date** 列包含的值早于 2016 年 1 月 1 日的行。


    -- Filter by date
    --
    CREATE FUNCTION dbo.fn_stretch_by_date(@date datetime2)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
           RETURN SELECT 1 AS is_eligible WHERE @date < CONVERT(datetime2, '1/1/2016', 101)
    GO


## 按状态列中的值筛选行
以下示例将迁移 **status** 列包含指定值之一的行。


    -- Filter by status column
    --
    CREATE FUNCTION dbo.fn_stretch_by_status(@status nvarchar(128))
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
           RETURN SELECT 1 AS is_eligible WHERE @status IN (N'Completed', N'Returned', N'Cancelled')
    GO


## 使用滑动窗口筛选行
若要使用滑动窗口筛选行，请记住筛选器函数的下列要求。

* 该函数必须是确定性函数。因此，不能创建随着时间的推移自动重新计算滑动窗口的函数。

* 该函数使用架构绑定。因此，不能只是通过调用 **ALTER FUNCTION** 移动滑动窗口来每天“就地”更新函数。

从类似于以下示例的筛选器函数开始，该函数将迁移 **systemEndTime** 列包含的值早于 2016 年 1 月 1 日的行。

    CREATE FUNCTION dbo.fn_StretchBySystemEndTime20160101(@systemEndTime datetime2)
    RETURNS TABLE
    WITH SCHEMABINDING  
    AS  
    RETURN SELECT 1 AS is_eligible
      WHERE @systemEndTime < CONVERT(datetime2, '2016-01-01T00:00:00', 101) ;

对表应用筛选器函数。

    ALTER TABLE <table name>
    SET (
            REMOTE_DATA_ARCHIVE = ON
                    (
                            FILTER_PREDICATE = dbo.fn_StretchBySystemEndTime20160101 (SysEndTime)
                                    , MIGRATION_STATE = OUTBOUND
                    )
            )
    ;

如果要更新滑动窗口，请执行以下操作。

1. 创建一个新函数，指定新的滑动窗口。以下示例选择的日期早于 2016 年 1 月 2 日，而不是 2016 年 1 月 1 日。

2. 通过调用 **ALTER TABLE** 将以前的筛选器函数替换为新筛选器函数，如以下示例所示。

3. （可选）通过调用 **DROP FUNCTION** 删除不再使用的以前的筛选器函数。（本示例中未说明此步骤。）


        BEGIN TRAN
        GO
            /*(1) Create new predicate function definition */
            CREATE FUNCTION dbo.fn_StretchBySystemEndTime20160102(@systemEndTime datetime2)
            RETURNS TABLE
            WITH SCHEMABINDING
            AS
            RETURN SELECT 1 AS is_eligible
                   WHERE @systemEndTime < CONVERT(datetime2,'2016-01-02T00:00:00', 101)
            GO
    
            /*(2) Set the new function as the filter predicate */
            ALTER TABLE <table name>
            SET
            (
                   REMOTE_DATA_ARCHIVE = ON
                   (
                           FILTER_PREDICATE = dbo.fn_StretchBySystemEndTime20160102(SysEndTime),
                           MIGRATION_STATE = OUTBOUND
                   )
            )
        COMMIT ;


## 有效的筛选器函数的更多示例

* 以下示例使用 AND 逻辑运算符组合两个基元条件。

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
    

* 以下示例使用了多个条件，并使用 CONVERT 执行确定性转换。

        CREATE FUNCTION dbo.fn_stretchpredicate_example1(@column1 datetime, @column2 int, @column3 nvarchar)
        RETURNS TABLE
        WITH SCHEMABINDING
        AS
        RETURN	SELECT 1 AS is_eligible
            WHERE @column1 < CONVERT(datetime, '1/1/2015', 101)AND (@column2 < -100 OR @column2 > 100 OR @column2 IS NULL)AND @column3 IN (N'Completed', N'Returned', N'Cancelled')
        GO
    

* 以下示例使用数学运算符和函数。

    
        CREATE FUNCTION dbo.fn_stretchpredicate_example2(@column1 float)
        RETURNS TABLE
        WITH SCHEMABINDING
        AS
        RETURN	SELECT 1 AS is_eligible
        		WHERE @column1 < SQRT(400) + 10
        GO
    

* 以下示例使用 BETWEEN 和 NOT BETWEEN 运算符。这种用法是有效的，因为在将 BETWEEN 和 NOT BETWEEN 运算符替换为等效的 AND 和 OR 表达式后，生成的函数符合本文所述的规则。

        CREATE FUNCTION dbo.fn_stretchpredicate_example3(@column1 int, @column2 int)
        RETURNS TABLE
        WITH SCHEMABINDING
        AS
        RETURN	SELECT 1 AS is_eligible
        		WHERE @column1 BETWEEN 0 AND 100
        			AND (@column2 NOT BETWEEN 200 AND 300 OR @column1 = 50)
        GO
    
    在将 BETWEEN 和 NOT BETWEEN 运算符替换为等效的 AND 和 OR 表达式后，上面的函数等效于下面的函数。

    
        CREATE FUNCTION dbo.fn_stretchpredicate_example4(@column1 int, @column2 int)
        RETURNS TABLE
        WITH SCHEMABINDING
        AS
        RETURN	SELECT 1 AS is_eligible
        		WHERE @column1 >= 0 AND @column1 <= 100AND (@column2 < 200 OR @column2 > 300 OR @column1 = 50)
        GO
    

## 无效的筛选器函数的示例

* 以下函数无效，因为它包含不确定性转换。

        CREATE FUNCTION dbo.fn_example5(@column1 datetime)
        RETURNS TABLE
        WITH SCHEMABINDING
        AS
        RETURN	SELECT 1 AS is_eligible
        		WHERE @column1 < CONVERT(datetime, '1/1/2016')
        GO
    

* 以下函数无效，因为它包含不确定性函数调用。

        CREATE FUNCTION dbo.fn_example6(@column1 datetime)
        RETURNS TABLE
        WITH SCHEMABINDING
        AS
        RETURN	SELECT 1 AS is_eligible
        		WHERE @column1 < DATEADD(day, -60, GETDATE())
        GO

* 以下函数无效，因为它包含子查询。

        CREATE FUNCTION dbo.fn_example7(@column1 int)
        RETURNS TABLE
        WITH SCHEMABINDING
        AS
        RETURN	SELECT 1 AS is_eligible
        		WHERE @column1 IN (SELECT SupplierID FROM Supplier WHERE Status = 'Defunct'))
        GO

* 以下函数无效，因为使用代数运算符或内置函数的表达式在你定义函数时必须求值为常量。不能在代数表达式或函数调用中包含列引用。

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

* 以下函数无效，因为在将 BETWEEN 运算符替换为等效的 AND 表达式后，该函数将违反本文所述的规则。

        CREATE FUNCTION dbo.fn_example10(@column1 int, @column2 int)
        RETURNS TABLE
        WITH SCHEMABINDING
        AS
        RETURN	SELECT 1 AS is_eligible
        		WHERE (@column1 BETWEEN 1 AND 200 OR @column1 = 300) AND @column2 > 1000
        GO
    
    在将 BETWEEN 运算符替换为等效的 AND 表达式后，上面的函数等效于下面的函数。此函数无效，因为基元条件只能使用 OR 逻辑运算符。

        CREATE FUNCTION dbo.fn_example11(@column1 int, @column2 int)
        RETURNS TABLE
        WITH SCHEMABINDING
        AS
        RETURN	SELECT 1 AS is_eligible
        		WHERE (@column1 >= 1 AND @column1 <= 200 OR @column1 = 300) AND @column2 > 1000
        GO

## Stretch Database 如何应用筛选器函数
Stretch Database 使用 CROSS APPLY 运算符对表应用筛选器函数并确定符合条件的行。例如：

    SELECT * FROM stretch_table_name CROSS APPLY fn_stretchpredicate(column1, column2)

如果该函数返回行的非空结果，则表示该行符合迁移条件。

## <a name="replacePredicate"></a>替换现有的筛选器函数
可以通过再次运行 **ALTER TABLE** 语句并为 **FILTER\_PREDICATE** 参数指定新值，来替换以前指定的筛选器函数。例如：

    ALTER TABLE stretch_table_name SET ( REMOTE_DATA_ARCHIVE = ON (
    	FILTER_PREDICATE = dbo.fn_stretchpredicate2(column1, column2),
    	MIGRATION_STATE = <desired_migration_state>

新的内联表值函数具有以下要求。

* 新函数的限制强度必须低于以前的函数。

* 旧函数中的所有运算符必须存在于新函数中。

* 新函数不能包含旧函数中不存在的运算符。

* 运算符参数的顺序不能更改。

* 只能更改 `<, <=, >, >=` 比较表达式中的常量值，但更改的方式必须使函数的限制性更低。

### 有效替换的示例
假设以下函数是当前的筛选器谓词。

    CREATE FUNCTION dbo.fn_stretchpredicate_old (@column1 datetime, @column2 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 < CONVERT(datetime, '1/1/2016', 101)
    			AND (@column2 < -100 OR @column2 > 100)
    GO

以下函数是有效的替换，因为新日期常量（指定一个更迟的截止日期）会使该函数的限制强度更低。

    CREATE FUNCTION dbo.fn_stretchpredicate_new (@column1 datetime, @column2 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 < CONVERT(datetime, '2/1/2016', 101)
    			AND (@column2 < -50 OR @column2 > 50)
    GO


### 无效替换的示例
以下函数不是有效的替换，因为新日期常量（指定一个更早的截止日期）不能使该函数的限制强度更低。

    CREATE FUNCTION dbo.fn_notvalidreplacement_1 (@column1 datetime, @column2 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 < CONVERT(datetime, '1/1/2015', 101)
    			AND (@column2 < -100 OR @column2 > 100)
    GO

以下函数不是有效的替换，因为删除了某个比较运算符。

    CREATE FUNCTION dbo.fn_notvalidreplacement_2 (@column1 datetime, @column2 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 < CONVERT(datetime, '1/1/2016', 101)
    			AND (@column2 < -50)
    GO

以下函数不是有效的替换，因为使用 AND 逻辑运算符添加了新的条件。

    CREATE FUNCTION dbo.fn_notvalidreplacement_3 (@column1 datetime, @column2 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 < CONVERT(datetime, '1/1/2016', 101)
    			AND (@column2 < -100 OR @column2 > 100)
    			AND (@column2 <> 0)
    GO

## 从表中删除筛选器函数
若要迁移整个表而不是选定的行，可以通过将 **FILTER\_PREDICATE** 设置为 null 来删除现有函数。例如：

    ALTER TABLE stretch_table_name SET ( REMOTE_DATA_ARCHIVE = ON (
    	FILTER_PREDICATE = NULL,
    	MIGRATION_STATE = <desired_migration_state>
    ) )

删除筛选器函数后，表中的所有行都符合迁移条件。因此，在后来不能为同一个表指定筛选器函数，除非先从 Azure 取回该表的所有远程数据。此限制的存在就是为了避免这样的情况：当提供新的筛选器函数时，不符合迁移条件的行已迁移到 Azure。

## 检查应用于表的筛选器函数
若要检查应用于表的筛选器函数，请打开目录视图 **sys.remote\_data\_archive\_tables** 并检查 **filter\_predicate** 列的值。如果值为 null，则整个表符合存档条件。有关详细信息，请参阅 [sys.remote\_data\_archive\_tables (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/dn935003.aspx)。

## 筛选器函数的安全注意事项  
具有 db\_owner 特权的盗用帐户可以执行以下操作。

* 创建并应用这样的表值函数：占用大量的服务器资源，或者等待较长时间从而导致拒绝服务。

* 创建并应用这样的表值函数：可以推导出该用户已被显式拒绝读取访问的表的内容。

## 另请参阅

[ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms190273.aspx)

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description:update meta properties;wording update-->