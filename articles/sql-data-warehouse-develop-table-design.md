<properties
   pageTitle="SQL 数据仓库中的表设计 | Azure"
   description="有关在开发解决方案时于 Azure SQL 数据仓库中设计表的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/14/2016"
   wacn.date="06/20/2016"/>

# SQL 数据仓库中的表设计 #
SQL 数据仓库是一种大规模并行处理 (MPP) 分布式数据库系统。它将数据存储在许多不同的位置（称为**分布区**）。每个**分布区**类似于一个桶，可在数据仓库中存储唯一的数据子集。通过将数据和处理能力分布于多个节点，SQL 数据仓库能够提供极大的缩放性 — 远超任何单一系统。

在 SQL 数据仓库中创建表时，表实际上分布在所有的分布区。

本文包括以下主题：

- 支持的数据类型
- 数据分布原则
- 轮循机制分布
- 哈希分布
- 表分区
- 统计信息
- 不支持的功能

## 支持的数据类型
SQL 数据仓库支持常见的业务数据类型：

- **bigint**
- **binary**
- **bit**
- **char**
- **date**
- **datetime**
- **datetime2**
- **datetimeoffset**
- **decimal**
- **float**
- **int**
- **money**
- **nchar**
- **nvarchar**
- **real**
- **smalldatetime**
- **smallint**
- **smallmoney**
- **sysname**
- **time**
- **tinyint**
- **uniqueidentifier**
- **varbinary**
- **varchar**

你可以使用以下查询来识别数据仓库中包含不兼容类型的列：

```
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

```

该查询包含任何不支持的用户定义数据类型。

以下是一些可用于取代不支持的数据类型的替代方案。

不要使用：

- **geometry**，而要使用 varbinary 类型
- **geography**，而要使用 varbinary 类型
- **hierarchyid**，CLR 类型不是本机的
- **image**、**text**、**ntext**（基于文本），而要使用 varchar/nvarchar（越小越好）
- **nvarchar(max)**，而要使用 nvarchar(4000) 或更小，以改善性能
- **numeric**，而要使用 decimal
- **sql\_variant**，将列拆分成多个强类型化列
- **sysname**，而要使用 nvarchar(128)
- **table**，转换成暂时表
- **timestamp**，修改代码以使用 datetime2 和 `CURRENT_TIMESTAMP` 函数。请注意，不能使用 current\_timestamp 作为默认约束，因为值不会自动更新。如果需要从 timestamp 类型化列迁移 rowversion 值，请对 NOT NULL 或 NULL 行版本值使用 BINARY(8) 或 VARBINARY(8)。
- **varchar(max)**，而要使用 varchar(8000) 或更小，以改善性能
- **uniqueidentifier**，而要使用 varbinary(8)
- **用户定义的类型**，尽可能转换回本机类型
- **xml**，而要使用 varchar(8000) 或更小，以改善性能 - 根据需要在列间拆分

部分支持：

- 默认约束仅支持文本和常量。不支持非确定性表达式或函数，例如 `GETDATE()` 或 `CURRENT_TIMESTAMP`。

> [AZURE.NOTE] 定义表，使最大可能的行大小（包括可变长度列的完整长度）不超过 32,767 个字节。虽然定义的行可以包含超过此数据的可变长度数据，但数据将无法插入表。此外，还请限制可变长度列的大小，以便运行查询时有更大的吞吐量。

## 数据分布原则

有两个选项可在 SQL 数据仓库中分布数据：

1. 均衡随机分布数据 
2. 根据单个列中的哈希值分布数据

在表级别决定数据分布。分布所有表。在 SQL 数据仓库数据库中分配每个表的分布。

第一个选项就是所谓的**轮询机制**分布 - 有时称为随机哈希。可以将此选项视为默认或防故障选项。

第二个选项称为**哈希**分布。可以将它视为数据分布的优化形式。当表群集共享通用的联接和/或聚合条件时，经常使用此选项。

## 轮循机制分布

轮循机制分布是尽可能让数据平均分布在所有分布区中的方法。包含行的缓冲区依次分配（因此名为轮循机制）到每个分布。该过程将不断重复，直到分配了所有数据缓冲区为止。数据不会在轮循机制分布式表中进行排序。出于此原因，轮循机制分布有时称为随机哈希。数据尽可能地平均分布在所有分布区中。

以下是轮循机制分布式表的示例：

```
CREATE TABLE [dbo].[FactInternetSales]
(   [ProductKey]            int          NOT NULL
,   [OrderDateKey]          int          NOT NULL
,   [CustomerKey]           int          NOT NULL
,   [PromotionKey]          int          NOT NULL
,   [SalesOrderNumber]      nvarchar(20) NOT NULL
,   [OrderQuantity]         smallint     NOT NULL
,   [UnitPrice]             money        NOT NULL
,   [SalesAmount]           money        NOT NULL
)
WITH
(   CLUSTERED COLUMNSTORE INDEX
,   DISTRIBUTION = ROUND_ROBIN
)
;
```

以下同样是轮循机制分布式表的一个示例：

```
CREATE TABLE [dbo].[FactInternetSales]
(   [ProductKey]            int          NOT NULL
,   [OrderDateKey]          int          NOT NULL
,   [CustomerKey]           int          NOT NULL
,   [PromotionKey]          int          NOT NULL
,   [SalesOrderNumber]      nvarchar(20) NOT NULL
,   [OrderQuantity]         smallint     NOT NULL
,   [UnitPrice]             money        NOT NULL
,   [SalesAmount]           money        NOT NULL
)
WITH
(   CLUSTERED COLUMNSTORE INDEX
)
;
```

> [AZURE.NOTE] 请注意，上述第二个示例并未提到分布键。轮循机制是默认设置，因此不是绝对必要的。但是，最好是明确分布方式，因为这可以在审查表设计时，确保你的同事知道你的意图。

没有明显的键列可哈希数据时，通常使用此表类型。较小或较不重要的表（其移动成本可能不是很高）也可以使用此表类型。

将数据载入轮循机制分布式表通常比载入哈希分布式表更快。使用轮循机制分布式表，加载前不需要了解数据或执行哈希。出于此原因，轮循机制表通常具有良好的加载目标。

> [AZURE.NOTE] 将数据进行轮循机制分布后，数据将在*缓冲区*级别分配到分布区。

### 建议

在以下情况下，请考虑对表使用轮循机制分布：

- 没有明显的联接键时
- 不知道候选哈希分布键时
- 表没有与其他表共享通用的联接键时
- 该联接比查询中的其他联接更不重要时
- 该表是初始加载表时

## 哈希分布

哈希分布使用内部函数通过哈希单个列来将数据集分布到各个分布区。哈希数据后，将数据分配到分布区不遵循明确的顺序。但是，哈希本身是确定性的过程。这使哈希结果变得可预测。例如，哈希包含值 10 的整数列时，始终会生成相同的哈希值。这意味着，***任何***包含值 10 的哈希整数列最终都会分配到相同的分布区。这一点适用于所有表。

哈希可预测性非常重要。也就是说，分布数据的哈希可能使读取数据和将表联接在一起时的性能得到改进。

如下所示，哈希分布对查询优化非常有用。这就是它为何被视为数据分布优化形式的原因。

> [AZURE.NOTE] 请记住！ 哈希不是基于数据值，而是基于要哈希的数据类型。

以下是根据 ProductKey 进行哈希分布的表。

```
CREATE TABLE [dbo].[FactInternetSales]
(   [ProductKey]            int          NOT NULL
,   [OrderDateKey]          int          NOT NULL
,   [CustomerKey]           int          NOT NULL
,   [PromotionKey]          int          NOT NULL
,   [SalesOrderNumber]      nvarchar(20) NOT NULL
,   [OrderQuantity]         smallint     NOT NULL
,   [UnitPrice]             money        NOT NULL
,   [SalesAmount]           money        NOT NULL
)
WITH
(   CLUSTERED COLUMNSTORE INDEX
,   DISTRIBUTION = HASH([ProductKey])
)
;
```

> [AZURE.NOTE] 将数据进行哈希分布后，数据将在行级别分配到分布区。

## 表分区
支持并且可轻松定义表分区。

SQL 数据仓库分区的 `CREATE TABLE` 命令示例：

```
CREATE TABLE [dbo].[FactInternetSales]
(
    [ProductKey]            int          NOT NULL
,   [OrderDateKey]          int          NOT NULL
,   [CustomerKey]           int          NOT NULL
,   [PromotionKey]          int          NOT NULL
,   [SalesOrderNumber]      nvarchar(20) NOT NULL
,   [OrderQuantity]         smallint     NOT NULL
,   [UnitPrice]             money        NOT NULL
,   [SalesAmount]           money        NOT NULL
)
WITH
(   CLUSTERED COLUMNSTORE INDEX
,   DISTRIBUTION = HASH([ProductKey])
,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                    (20000101,20010101,20020101
                    ,20030101,20040101,20050101
                    )
                )
)
;
```

请注意，定义中没有数据分区函数或方案。所有这些信息都是在创建表时处理。你只需识别即将成为分区键的列的边界点。

## 统计信息

当用户查询表时，SQL 数据仓库将使用分布式查询优化器来创建适当的查询计划。创建后，查询计划将提供数据库用于访问数据和履行用户请求的策略与方法。SQL 数据仓库的查询优化器基于成本。换句话说，它根据相对成本来比较各种选项（计划）并选择最有效率的可用计划。因此，SQL 数据仓库需要大量信息才能做出基于成本的明智决策。它保留有关表（表大小）的统计信息，在数据库对象中称为 `STATISTICS`。

统计信息针对索引或表的单个或多个列保留。这些数据可将值的基数和选择性相关重要信息提供给基于成本的优化器。当优化器需要评估查询中的 JOIN、GROUP BY、HAVING 和 WHERE 子句时，这些数据特别有用。因此，这些统计信息对象包含的信息一定要*准确*反映表的当前状态。请务必知道，成本的准确性非常重要。如果统计信息准确反映了表的状态，则可以比较计划以找出最低的成本。如果统计信息不准确，SQL 数据仓库可能会选择错误的计划。

SQL 数据仓库中的列级统计信息由用户定义。

换句话说，我们必须自行创建这些统计信息。我们刚才知道，这件事不容忽视。这是 SQL Server 与 SQL 数据仓库间的重要差异。查询列时，SQL Server 会自动创建统计信息。默认情况下，SQL Server 还会自动更新这些统计信息。但是，在 SQL 数据仓库中，需要手动创建和管理统计信息。

### 建议

生成统计信息时，请遵循以下建议：

1. 对 `WHERE`、`JOIN`、`GROUP BY`、`ORDER BY` 和 `DISTINCT` 子句使用的列，请创建单列统计信息
2. 生成复合子句的多列统计信息
3. 定期更新统计信息。请记住，此操作不会自动完成！

>[AZURE.NOTE] SQL Server 数据仓库通常完全依赖 `AUTOSTATS` 来保持列统计信息的最新状态。对于 SQL Server 数据仓库而言，这不是最佳实践。20% 的变化率将触发 `AUTOSTATS`，这对于包含数百万或数十亿行的大型事实表而言可能不太足够。因此，最好随时掌握统计信息更新，以确保统计信息能准确地反映表的基数。

## 不支持的功能
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


## 后续步骤
有关更多开发技巧，请参阅[开发概述][]。

<!--Image references-->

<!--Article references-->
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/

<!--MSDN references-->

<!--Other Web references-->

<!---HONumber=Mooncake_0321_2016-->