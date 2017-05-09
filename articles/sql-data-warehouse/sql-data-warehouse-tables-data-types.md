<!-- Temp remove tables-overview, next task on -->
<properties
    pageTitle="SQL 数据仓库中表的数据类型 | Azure"
    description="Azure SQL 数据仓库表的数据类型入门。"
    services="sql-data-warehouse"
    documentationCenter="NA"
    author="jrowlandjones"
    manager="barbkess"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="d4a1f0a3-ba9f-44b9-95f6-16a4f30746d6"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.custom="tables"
    ms.date="10/31/2016"
    wacn.date="05/08/2017"
    ms.author="jrj;barbkess"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="8dcf78dc932010877830130ba29f3f7f54d6c925"
    ms.lasthandoff="04/28/2017" />

# <a name="data-types-for-tables-in-sql-data-warehouse"></a>SQL 数据仓库中表的数据类型
> [AZURE.SELECTOR]
> * [概述][Overview]
> * [数据类型][Data Types]
> * [分布][Distribute]
> * [索引][Index]
> * [分区][Partition]
> * [统计信息][Statistics]
> * [临时][Temporary]
> 
> 

SQL 数据仓库支持最常用的数据类型。  下面是 SQL 数据仓库支持的数据类型列表。  有关支持的数据类型的详细信息，请参阅 [创建表][create table]。

| **支持的数据类型** |  |  |
| --- | --- | --- |
| [bigint][bigint] |[decimal][decimal] |[smallint][smallint] |
| [binary][binary] |[float][float] |[smallmoney][smallmoney] |
| [bit][bit] |[int][int] |[sysname][sysname] |
| [char][char] |[money][money] |[time][time] |
| [日期][date] |[nchar][nchar] |[tinyint][tinyint] |
| [datetime][datetime] |[nvarchar][nvarchar] |[uniqueidentifier][uniqueidentifier] |
| [datetime2][datetime2] |[real][real] |[varbinary][varbinary] |
| [datetimeoffset][datetimeoffset] |[smalldatetime][smalldatetime] |[varchar][varchar] |

## <a name="data-type-best-practices"></a>数据类型最佳实践
 在定义列类型时，使用可支持数据的最小数据类型，将能够改善查询性能。 这对 CHAR 和 VARCHAR 列尤其重要。 如果列中最长的值是 25 个字符，请将列定义为 VARCHAR(25)。 避免将所有字符列定义为较大的默认长度。 此外，将列定义为 VARCHAR（当它只需要这样的大小时）而非 [NVARCHAR][NVARCHAR]。  尽可能使用 NVARCHAR(4000) 或 VARCHAR(8000)，而非 NVARCHAR(MAX) 或 VARCHAR(MAX)。

## <a name="polybase-limitation"></a>Polybase 限制
如果使用 Polybase 加载表，请确保数据的长度不超过 1 MB。  虽然你在定义行时可以使用超出此宽度的可变长度数据，并通过 BCP 来加载行，但无法使用 Polybase 来加载此数据。  

## <a name="unsupported-data-types"></a>不支持的数据类型
如果从另一个 SQL 平台（例如 Azure SQL 数据库）迁移数据库，在迁移时，你可能会遇到 SQL 数据仓库不支持的某些数据类型。  下面是不支持的数据类型，以及一些可用于取代不支持的数据类型的备选项。

| 数据类型 | 解决方法 |
| --- | --- |
| [geometry][geometry] |[varbinary][varbinary] |
| [geography][geography] |[varbinary][varbinary] |
| [hierarchyid][hierarchyid] |[nvarchar][nvarchar](4000) |
| [image][ntext,text,image] |[varbinary][varbinary] |
| [text][ntext,text,image] |[varchar][varchar] |
| [ntext][ntext,text,image] |[nvarchar][nvarchar] |
| [sql_variant][sql_variant] |将列拆分成多个强类型化列。 |
| [表][table] |转换成暂时表。 |
| [timestamp][timestamp] |修改代码以使用 [datetime2][datetime2] 和 `CURRENT_TIMESTAMP` 函数。  仅支持常量作为默认值，因此，不能将 current_timestamp 定义为默认约束。 如果需要从 timestamp 类型化列迁移行版本值，请对 NOT NULL 或 NULL 行版本值使用 [BINARY][BINARY](8) 或 [VARBINARY][BINARY](8)。 |
| [xml][xml] |[varchar][varchar] |
| [用户定义的类型][user defined types] |尽可能转换回本机类型 |
| 默认值 |默认值仅支持文本和常量。  不支持非确定性表达式或函数，例如 `GETDATE()` 或 `CURRENT_TIMESTAMP`。 |

可以在当前 SQL 数据库上运行以下 SQL 来识别 Azure SQL 数据仓库不支持的列：

    SELECT  t.[name], c.[name], c.[system_type_id], c.[user_type_id], y.[is_user_defined], y.[name]
    FROM sys.tables  t
    JOIN sys.columns c on t.[object_id]    = c.[object_id]
    JOIN sys.types   y on c.[user_type_id] = y.[user_type_id]
    WHERE y.[name] IN ('geography','geometry','hierarchyid','image','text','ntext','sql_variant','timestamp','xml')
    AND  y.[is_user_defined] = 1;

## <a name="next-steps"></a>后续步骤
有关详细信息，请参阅有关[表概述][Overview]、[分布表][Distribute]、[为表编制索引][Index]、[将表分区][Partition]、维[护表统计信息][Statistics]和[临时表][Temporary]的文章。  有关最佳实践的详细信息，请参阅 [SQL 数据仓库最佳实践][SQL Data Warehouse Best Practices]。

<!--Image references-->

<!--Article references-->
[Overview]: /documentation/articles/sql-data-warehouse-tables-overview/
[Data Types]: /documentation/articles/sql-data-warehouse-tables-data-types/
[Distribute]: /documentation/articles/sql-data-warehouse-tables-distribute/
[Index]: /documentation/articles/sql-data-warehouse-tables-index/
[Partition]: /documentation/articles/sql-data-warehouse-tables-partition/
[Statistics]: /documentation/articles/sql-data-warehouse-tables-statistics/
[Temporary]: /documentation/articles/sql-data-warehouse-tables-temporary/
[SQL Data Warehouse Best Practices]: /documentation/articles/sql-data-warehouse-best-practices/

<!--MSDN references-->

<!--Other Web references-->
[create table]: https://msdn.microsoft.com/zh-cn/library/mt203953.aspx
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

<!--Update_Description:update meta properties;wording update-->