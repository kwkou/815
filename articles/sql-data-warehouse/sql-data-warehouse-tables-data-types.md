<!-- Temp remove tables-overview, next task on -->
<properties
   pageTitle="SQL 数据仓库中表的数据类型 | Azure"
   description="Azure SQL 数据仓库表的数据类型入门。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="10/31/2016"
   wacn.date="12/19/2016"
   ms.author="jrj;barbkess;sonyama"/>

# SQL 数据仓库中表的数据类型

> [AZURE.SELECTOR]
- [数据类型][]
- [分布][]
- [索引][]
<!-- 
- [概述][]
- [Partition][]
- [统计信息][]
- [临时][]
-->

SQL 数据仓库支持最常用的数据类型。下面是 SQL 数据仓库支持的数据类型列表。有关支持的数据类型的详细信息，请参阅[创建表][]。

|**支持的数据类型**|    |    |
|---|---|---|
|[bigint][]|[decimal][]|[smallint][]|
|[binary][]|[float][]|[smallmoney][]|
|[bit][]|[int][]|[sysname][]|
|[char][]|[money][]|[time][]|
|[date][]|[nchar][]|[tinyint][]|
|[datetime][]|[nvarchar][]|[uniqueidentifier][]|
|[datetime2][]|[real][]|[varbinary][]|
|[datetimeoffset][]|[smalldatetime][]|[varchar][]|


## 数据类型最佳实践

 在定义列类型时，使用可支持数据的最小数据类型，将能够改善查询性能。这对 CHAR 和 VARCHAR 列尤其重要。如果列中最长的值是 25 个字符，请将列定义为 VARCHAR(25)。避免将所有字符列定义为较大的默认长度。此外，将列定义为 VARCHAR（当它只需要这样的大小时）而非 [NVARCHAR][]。尽可能使用 NVARCHAR(4000) 或 VARCHAR(8000)，而非 NVARCHAR(MAX) 或 VARCHAR(MAX)。

## Polybase 限制

如果你是使用 Polybase 来加载表，则可对表进行定义，使可能的最大行大小（包括可变长度列的完整长度）不超过 32,767 字节。虽然你在定义行时可以使用超出此宽度的可变长度数据，并通过 BCP 来加载行，但无法使用 Polybase 来加载此数据。很快会增加针对宽行的 Polybase 支持。

## 不支持的数据类型

如果从另一个 SQL 平台（例如 Azure SQL 数据库）迁移数据库，在迁移时，你可能会遇到 SQL 数据仓库不支持的某些数据类型。下面是不支持的数据类型，以及一些可用于取代不支持的数据类型的备选项。

|数据类型|解决方法|
|---|---|
|[geometry][]|[varbinary][]|
|[geography][]|[varbinary][]|
|[hierarchyid][]|[nvarchar][](4000)|
|[geography][ntext,text,image]|[varbinary][]|
|[text][ntext,text,image]|[varchar][]|
|[ntext][ntext,text,image]|[nvarchar][]|
|[sql_variant][]|将列拆分成多个强类型化列。|
|[table][]|转换成暂时表。|
|[timestamp][]|修改代码以使用 [datetime2][] 和 **CURRENT_TIMESTAMP** 函数。仅支持常量作为默认值，因此，不能将 current_timestamp 定义为默认约束。如果需要从 timestamp 类型化列迁移 rowversion 值，请对 NOT NULL 或 NULL 行版本值使用 [BINARY][](8) 或 [VARBINARY][BINARY](8)。|
|[xml][]|[varchar][]|
|[user defined types][]|尽可能转换回本机类型|
|默认值|默认值仅支持文本和常量。不支持非确定性表达式或函数，例如 **GETDATE()** 或 **CURRENT_TIMESTAMP**。|

可以在当前 SQL 数据库上运行以下 SQL 来识别 Azure SQL 数据仓库不支持的列：


    SELECT  t.[name], c.[name], c.[system_type_id], c.[user_type_id], y.[is_user_defined], y.[name]
    FROM sys.tables  t
    JOIN sys.columns c on t.[object_id]    = c.[object_id]
    JOIN sys.types   y on c.[user_type_id] = y.[user_type_id]
    WHERE y.[name] IN ('geography','geometry','hierarchyid','image','text','ntext','sql_variant','timestamp','xml')
    AND  y.[is_user_defined] = 1;


## 后续步骤

有关详细信息，请参阅有关 <!-- [表概述][Overview]、--> [分布表][Distribute]、[为表编制索引][Index] <!-- 、[将表分区][Partition]、[维护表统计信息][Statistics]和[临时表][Temporary] --> 的文章。有关最佳实践详细信息，请参阅 [SQL Data Warehouse Best Practices][]（SQL 数据仓库最佳实践）。

<!--Image references-->

<!--Article references-->
[Overview]: /documentation/articles/sql-data-warehouse-tables-overview/
[概述]: /documentation/articles/sql-data-warehouse-tables-overview/
[数据类型]: /documentation/articles/sql-data-warehouse-tables-data-types/
[Distribute]: /documentation/articles/sql-data-warehouse-tables-distribute/
[分布]: /documentation/articles/sql-data-warehouse-tables-distribute/
[Index]: /documentation/articles/sql-data-warehouse-tables-index/
[索引]: /documentation/articles/sql-data-warehouse-tables-index/
[Partition]: /documentation/articles/sql-data-warehouse-tables-partition/
[Statistics]: /documentation/articles/sql-data-warehouse-tables-statistics/
[统计信息]: /documentation/articles/sql-data-warehouse-tables-statistics/
[Temporary]: /documentation/articles/sql-data-warehouse-tables-temporary/
[临时]: /documentation/articles/sql-data-warehouse-tables-temporary/
[SQL Data Warehouse Best Practices]: /documentation/articles/sql-data-warehouse-best-practices/

<!--MSDN references-->

<!--Other Web references-->
[创建表]: https://msdn.microsoft.com/zh-cn/library/mt203953.aspx
[bigint]: https://msdn.microsoft.com/zh-cn/library/ms187745.aspx
[binary]: https://msdn.microsoft.com/zh-cn/library/ms188362.aspx
[bit]: https://msdn.microsoft.com/zh-cn/library/ms177603.aspx
[char]: https://msdn.microsoft.com/zh-cn/library/ms176089.aspx
[date]: https://msdn.microsoft.com/zh-cn/library/bb630352.aspx
[datetime]: https://msdn.microsoft.com/zh-cn/library/ms187819.aspx
[datetime2]: https://msdn.microsoft.com/zh-cn/library/bb677335.aspx
[datetimeoffset]: https://msdn.microsoft.com/zh-cn/library/bb630289.aspx
[decimal]: https://msdn.microsoft.com/zh-cn/library/ms187746.aspx
[float]: https://msdn.microsoft.com/zh-cn/library/ms173773.aspx
[geometry]: https://msdn.microsoft.com/zh-cn/library/cc280487.aspx
[geography]: https://msdn.microsoft.com/zh-cn/library/cc280766.aspx
[hierarchyid]: https://msdn.microsoft.com/zh-cn/library/bb677290.aspx
[int]: https://msdn.microsoft.com/zh-cn/library/ms187745.aspx
[money]: https://msdn.microsoft.com/zh-cn/library/ms179882.aspx
[nchar]: https://msdn.microsoft.com/zh-cn/library/ms186939.aspx
[nvarchar]: https://msdn.microsoft.com/zh-cn/library/ms186939.aspx
[ntext,text,image]: https://msdn.microsoft.com/zh-cn/library/ms187993.aspx
[real]: https://msdn.microsoft.com/zh-cn/library/ms173773.aspx
[smalldatetime]: https://msdn.microsoft.com/zh-cn/library/ms182418.aspx
[smallint]: https://msdn.microsoft.com/zh-cn/library/ms187745.aspx
[smallmoney]: https://msdn.microsoft.com/zh-cn/library/ms179882.aspx
[sql_variant]: https://msdn.microsoft.com/zh-cn/library/ms173829.aspx
[sysname]: https://msdn.microsoft.com/zh-cn/library/ms186939.aspx
[table]: https://msdn.microsoft.com/zh-cn/library/ms175010.aspx
[time]: https://msdn.microsoft.com/zh-cn/library/bb677243.aspx
[timestamp]: https://msdn.microsoft.com/zh-cn/library/ms182776.aspx
[tinyint]: https://msdn.microsoft.com/zh-cn/library/ms187745.aspx
[uniqueidentifier]: https://msdn.microsoft.com/zh-cn/library/ms187942.aspx
[varbinary]: https://msdn.microsoft.com/zh-cn/library/ms188362.aspx
[varchar]: https://msdn.microsoft.com/zh-cn/library/ms186939.aspx
[xml]: https://msdn.microsoft.com/zh-cn/library/ms187339.aspx
[user defined types]: https://msdn.microsoft.com/zh-cn/library/ms131694.aspx

<!---HONumber=Mooncake_1212_2016-->