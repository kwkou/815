<properties
   pageTitle="将架构迁移到 SQL 数据仓库 | Azure"
   description="有关在开发解决方案时将架构迁移到 Azure SQL 数据仓库的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/14/2016"
   wacn.date="06/20/2016"/>

# 将架构迁移到 SQL 数据仓库#

以下摘要将帮助你理解 SQL Server 与 SQL 数据仓库之间的差异，以便你迁移数据库。

### 表功能
SQL 数据仓库不使用或不支持以下功能：

- 主键  
- 外键  
- 检查约束  
- 唯一约束  
- 唯一索引  
- 计算列  
- 稀疏列  
- 用户定义的类型  
- 索引视图  
- 标识  
- 序列  
- 触发器  
- 同义词  


### 数据类型差异
SQL 数据仓库支持常见的业务数据类型：

- bigint
- binary
- bit
- char
- 日期
- datetime
- datetime2
- datetimeoffset
- decimal
- float
- int
- money
- nchar
- nvarchar
- numeric
- real
- smalldatetime
- smallint
- smallmoney
- sysname
- time
- tinyint
- varbinary
- varchar
- uniqueidentifier

你可以使用此查询来识别数据仓库中包含不兼容类型的列：

    SELECT  t.[name]
    ,       c.[name]
    ,       c.[system_type_id]
    ,       c.[user_type_id]
    ,       y.[is_user_defined]
    ,       y.[name]
    FROM sys.tables  t
    JOIN sys.columns c on t.[object_id]    = c.[object_id]
    JOIN sys.types   y on c.[user_type_id] = y.[user_type_id]
    WHERE y.[name] IN
                    (   'geography'
                    ,   'geometry'
                    ,   'hierarchyid'
                    ,   'image'
                    ,   'ntext'
                    ,   'sql_variant'
                    ,   'text'
                    ,   'timestamp'
                    ,   'xml'
                    )
    AND  y.[is_user_defined] = 1
    ;

该查询包含同样不受支持的任何用户定义数据类型。

如果你的数据库包含不受支持的类型，请不要担心。下面推荐了一些可以使用的替代方案。

不要使用：

- **geometry**，而要使用 varbinary 类型
- **geography**，而要使用 varbinary 类型
- **hierarchyid**，此 CLR 类型不受支持
- **image**、**text**、**ntext**，而要使用 varchar/nvarchar（越小越好）
- **sql\_variant**，将列拆分成多个强类型化列
- **table**，转换成暂时表
- **timestamp**，修改代码以使用 datetime2 和 `CURRENT_TIMESTAMP` 函数。请注意，不能使用 current\_timestamp 作为默认约束，因为值不会自动更新。如果需要从 timestamp 类型化列迁移 rowversion 值，请对 NOT NULL 或 NULL 行版本值使用 binary(8) 或 varbinary(8)。
- **用户定义的类型**，尽可能转换回本机类型
- **xml**，而要使用 varchar(max) 或更小，以改善性能。根据需要在列之间拆分

为了提高性能，不要使用：

- nvarchar(max)，而要使用 nvarchar(4000) 或更小，以改善性能
- varchar(max)，而要使用 varchar(8000) 或更小，以改善性能

部分支持：

- 默认约束仅支持文本和常量。不支持非确定性表达式或函数，例如 `GETDATE()` 或 `CURRENT_TIMESTAMP`。

> [AZURE.NOTE] 如果你是使用 Polybase 来加载表，则可对表进行定义，使可能的最大行大小（包括可变长度列的完整长度）不超过 32,767 字节。虽然你在定义行时可以使用超出此数字的可变长度数据，并通过 BCP 来加载行，但目前仍无法使用 Polybase 来加载此数据。很快会增加针对宽行的 Polybase 支持。此外，还请限制可变长度列的大小，以便运行查询时有更大的吞吐量。

## 后续步骤
成功将数据库架构迁移到 SQL 数据仓库后，可以继续阅读下列文章之一：

- [迁移数据][]
- [迁移代码][]

有关更多开发技巧，请参阅[开发概述][]。

<!--Image references-->

<!--Article references-->
[迁移代码]: /documentation/articles/sql-data-warehouse-migrate-code/
[迁移数据]: /documentation/articles/sql-data-warehouse-migrate-data/
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/

<!--MSDN references-->


<!--Other Web references-->

<!---HONumber=Mooncake_0613_2016-->