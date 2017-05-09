<properties
    pageTitle="提高 Azure SQL 中的列存储索引性能 | Azure"
    description="减少内存需求或增加可用内存，使列存储索引压缩到每个行组中的行数最大化。"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="shivaniguptamsft"
    manager="jhubbard"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="ef170f39-ae24-4b04-af76-53bb4c4d16d3"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.custom="performance"
    ms.date="11/18/2016"
    wacn.date="05/08/2017"
    ms.author="shigu;barbkess"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="e08c70926c46bb78d92e741d05c7d36506f76aee"
    ms.lasthandoff="04/28/2017" />

# <a name="memory-optimizations-for-columnstore-compression"></a>列存储压缩的内存优化

减少内存需求或增加可用内存，使列存储索引压缩到每个行组中的行数最大化。 使用这些方法来提高列存储索引的压缩率和请求性能。

## <a name="why-the-rowgroup-size-matters"></a>行组大小之所以重要的原因
由于列存储索引会通过扫描单个行组的列段来扫描表，所以，使每个行组的行数最大化可增强查询性能。 如果行组具有的行数较多，则会增强数据压缩，这意味着需要从磁盘读取的数据变少。

有关行组的详细信息，请参阅[列存储索引指南](https://msdn.microsoft.com/zh-cn/library/gg492088.aspx)。

## <a name="target-size-for-rowgroups"></a>行组的目标大小
为了获得最佳性能，需要使列存储索引中每个行组的行数最大化。一个行组最多可有 1,048,576 个行。每个行组不一定要具有最大行数。行组至少有 100,000 行时，列存储索引可获得良好的性能。

## <a name="rowgroups-can-get-trimmed-during-compression"></a>在压缩过程中，可对行组进行修剪

批量加载或重建列存储索引期间，有时可能因内存不足而无法压缩为每个行组指定的所有行。 如果出现内存压力，列存储索引将修剪行组大小，以便能成功将行组压缩到列存储中。

如果内存不足，无法将至少 10,000 个行压缩到每个行组中，SQL 数据仓库将生成错误。

有关批量加载的详细信息，请参阅 [Bulk load into a clustered columnstore index](https://msdn.microsoft.com/zh-cn/library/dn935008.aspx#Bulk load into a clustered columnstore index)（批量加载到聚集列存储索引中）。

## 如何估算内存需求

压缩单个行组所需的最大内存大约为

- 72 MB +
- \#rows \* \#columns \* 8 字节 +
- \#rows \* \#short-string-columns \* 32 字节 +
- \#long-string-columns \* 16 MB 用于压缩字典

其中，short-string-columns 使用 <= 32 字节的字符串数据类型，long-string-columns 使用 > 32 字节的字符串数据类型。

使用专为压缩文本设计的压缩方法来压缩长字符串。此压缩方法使用*词典*来存储文本模式。词典最大大小为 16 MB。行组中每个长字符串列只能有一个词典。

有关列存储内存需求的深入讨论，请观看视频 [Azure SQL Data Warehouse scaling: configuration and guidance](https://myignite.microsoft.com/videos/14822)（Azure SQL 数据仓库缩放：配置和指南）。

## <a name="ways-to-reduce-memory-requirements"></a>减少内存需求的方法

使用以下技巧来减少内存需求，以便能将行组压缩到列存储索引中。

### <a name="use-fewer-columns"></a>减少所用列数
设计表时尽可能减少所用列数。如果行组已压缩到列存储中，列存储索引将单独压缩每个列段。因此，用于压缩行组的内存需求将随列数的增加而增加。

### <a name="use-fewer-string-columns"></a>减少字符串列数
字符串数据类型的列比数字和日期数据类型需要更多内存。若要减少内存需求，请考虑从事实表中删除字符串列，并将其置于维度较小的表中。

字符串压缩的额外内存需求：

- 对于最多 32 个字符的字符串数据类型，每个值可能需要 32 个额外字节。
- 具有超过 32 个字符的字符串数据类型会通过词典的方法来进行压缩。  行组中每个列可能需要最多 16 MB 的额外内存来生成词典。 

### <a name="avoid-over-partitioning"></a>避免过度分区

列存储索引会为每个分区创建一个或多个行组。 在 SQL 数据仓库中，由于将分发数据并将每次分发分区，因此分区数将快速增加。 如果表中分区过多，可能没有足够行来填充行组。 如果缺少行，在压缩过程中不会产生内存不足的情况，但是这会导致行组无法实现最佳列存储查询性能。

要避免过度分区的另一个原因是，在分区表上将行加载到列存储索引中会产生内存开销。 在加载期间，多数分区可能会收到传入行，它们会保存在内存中，直到每个分区都有足够行可进行压缩。 如果分区过多，可能产生额外的内存压力。 

### <a name="simplify-the-load-query"></a>简化加载查询

数据库会在查询的所有运算符之间共享查询的内存授予。 如果加载查询的排序和联接复杂，可用于压缩的内存将减少。

请仅针对加载查询而设计加载查询。 如果要对数据运行转换，请与加载查询分开来运行转换。 例如，将数据暂存在一个堆表中，运行转换，然后将临时表加载到列存储索引中。 也可先加载数据，然后使用 MPP 系统来转换数据。

### <a name="adjust-maxdop"></a>调整 MAXDOP

当每次分发有多个 CPU 内核可用时，每次分发会将行组并行压缩到列存储中。并行度需要额外的内存资源，这可能会造成内存压力和行组修剪。

若要减少内存压力，可使用 MAXDOP 查询提示，在每次分发中强制加载操作以串行模式运行。

    CREATE TABLE MyFactSalesQuota 
    WITH (DISTRIBUTION = ROUND_ROBIN)
    AS SELECT * FROM FactSalesQUota 
    OPTION (MAXDOP 1);

## <a name="ways-to-allocate-more-memory"></a>分配更多内存的方法

DWU 大小和用户资源类共同确定用户查询可用的内存量。 若要增加加载查询的内存授予，可增加 DWU 的数量或增加资源类。

- 若要增加 DWU，请参阅[如何进行性能缩放？](/documentation/articles/sql-data-warehouse-manage-compute-overview/#scale-compute)
- 若要更改查询的资源类，请参阅[更改用户资源类示例](/documentation/articles/sql-data-warehouse-develop-concurrency/#change-a-user-resource-class-example)。

例如，使用 DWU 100 时，smallrc 资源类中的用户在每次分发时可使用 100 MB 的内存。 有关详细信息，请参阅 [SQL 数据仓库中的并发](/documentation/articles/sql-data-warehouse-develop-concurrency/)。

假设需要 700 MB 的内存以获取优质行组大小。 以下示例演示可如何使用足够的内存来运行加载查询。

- 使用 DWU 1000 和 mediumrc，则内存授予为 800 MB
- 使用 DWU 600 和 largerc，则内存授予为 800 MB。

## <a name="next-steps"></a>后续步骤

若要了解提升 SQL 数据仓库中性能的更多方法，请参阅[性能概述](/documentation/articles/sql-data-warehouse-overview-manage-user-queries/)。

<!--Image references-->


<!--Article references-->


<!--MSDN references-->

<!--Other Web references-->

<!--Update_Description:update meta properties;wording update-->