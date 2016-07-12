<properties
   pageTitle="管理索引 | Azure"
   description="用以帮助用户管理其索引的指南"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/18/2016"
   wacn.date="04/25/2016"/>

# 管理索引
本文演示用于管理索引的一些关键技巧。

## 优化索引重新生成
可以通过以下两种方法中的一种来优化你的索引重新生成：

1. 对表进行分区并执行分区级别索引重新生成
2. 使用 [CTAS][] 在新表中重新创建数据的分区；然后在新表中进行分区切换

## 已分区的索引重新生成

下面为如何重新生成单个分区的示例：

```
ALTER INDEX ALL ON [dbo].[FactInternetSales] REBUILD Partition = 5
```

ALTER INDEX..REBUILD 最好用于较小的数据卷 - 尤其针对列存储索引。打开的、关闭的和压缩的行组全部包含在重新生成中。但是，如果分区相当大，那么你将发现 `CTAS` 的操作效率更高。下面为完整索引重新生成的示例

```
ALTER INDEX ALL ON [dbo].[DimProduct] REBUILD
```

请参阅 [ALTER INDEX][] 一文获取有关此语法的更多详细信息。

## 使用 CTAS 重新生成分区

下面为如何使用 CTAS 重新生成分区的示例：

```
-- Step 01. Select the partition of data and write it out to a new table using CTAS
CREATE TABLE [dbo].[FactInternetSales_20000101_20010101]
    WITH    (   DISTRIBUTION = HASH([ProductKey])
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101,20010101
                                )
                            )
            )
AS
SELECT  *
FROM    [dbo].[FactInternetSales]
WHERE   [OrderDateKey] >= 20000101
AND     [OrderDateKey] <  20010101
;

-- Step 02. Create a SWITCH out table
 
CREATE TABLE dbo.FactInternetSales_20000101
    WITH    (   DISTRIBUTION = HASH(ProductKey)
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101
                                )
                            )
            )
AS
SELECT *
FROM    [dbo].[FactInternetSales]
WHERE   1=2 -- Note this table will be empty

-- Step 03. Switch OUT the data 
ALTER TABLE [dbo].[FactInternetSales] SWITCH PARTITION 2 TO  [dbo].[FactInternetSales_20000101] PARTITION 2;

-- Step 04. Switch IN the rebuilt data
ALTER TABLE [dbo].[FactInternetSales_20000101_20010101] SWITCH PARTITION 2 TO  [dbo].[FactInternetSales] PARTITION 2;

```

## 后续步骤
有关如何创建分区和调整分区大小的更多指导，请参阅关于[表分区][]的文章。本文还包含一个用以帮助标识分区边界的示例。

有关更多管理提示，请转到[管理][]概述

<!--Image references-->

<!--Article references-->
[CTAS]: /documentation/articles/sql-data-warehouse-develop-ctas/
[表分区]: /documentation/articles/sql-data-warehouse-develop-table-partitions/
[管理]: /documentation/articles/sql-data-warehouse-manage-monitor/

<!--MSDN references-->
[ALTER INDEX]: https://msdn.microsoft.com/zh-cn/library/ms188388.aspx

<!--Other Web references-->
<!---HONumber=Mooncake_0418_2016-->