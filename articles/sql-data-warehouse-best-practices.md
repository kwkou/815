<!-- Remove data-factory -->
<properties
   pageTitle="Azure SQL 数据仓库最佳实践 | Azure"
   description="开发 Azure SQL 数据仓库解决方案时应了解的建议和最佳实践。这些内容可帮助你取得成功。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/02/2016"
   wacn.date="05/30/2016"/>

# SQL 数据仓库最佳实践

本文包含一系列最佳实践，可让你从 Azure SQL 数据仓库获得最佳价格/性能比。文章中有些概念非常基本且很容易解释，而有些概念则更高级，本文中只涉及了表面。本文的目的是要提供一些基本指导，以及让对创建数据仓库时的一些重要考虑因素有所认知。每部分都将介绍一个概念，并提供哪里可以阅读深度介绍的详细文章。

如果你刚开始使用 Azure SQL 数据仓库，千万别让本文吓到。主题的顺序主要按照重要性排列。如果你将重点放在前三个主题，一般不会有什么问题。当你更熟悉 SQL 数据仓库且能运用自如，请再回来看看其他主题。融会贯通不需要很长时间。一开始可能将有点不知所措，但经过一段时间和经验累积，这些主题对你来说将变得很基本。

## 使用暂停和缩放来降低成本
SQL 数据仓库的一个重要功能，是能够在不使用它时予以暂停，这将停止计算资源的计费。另一个重要功能是能够缩放资源。暂停和缩放可以通过  PowerShell 命令来完成。请熟悉这些功能，因为这些功能可以在数据仓库不使用时大幅降低成本。如果希望随时可访问数据仓库，建议将其缩放到最小的大小 (DW100)，而不是暂停。

另请参阅 [Pause compute resources][]（暂停计算资源）、[Resume compute resources][]（恢复计算资源）和 [Scale compute resources][]（缩放计算资源）。


## 暂停或缩放之前清空事务 
当暂停或缩放 SQL 数据仓库时，数据库实例将在幕后停止。这意味着所有进行中的查询都将取消。取消简单的 SELECT 查询是很快的操作，对于暂停或缩放实例所花费的时间几乎没有什么影响。但是，事务性查询（将修改数据或结构）可能无法快速地停止。**事务性查询必须完全完成或回滚更改。** 回滚事务性查询已完成的任务可能需要很长时间，甚至比查询应用原始更改更久。例如，如果取消的删除行查询已经运行一小时，系统可能需要一个小时重新插入已删除的行。如果在事务运行中运行暂停或缩放，暂停或缩放操作可能需要一些时间，因为暂停和缩放必须等回滚完成才能继续。

另请参阅 [Transactions in SQL Data Warehouse][]（SQL 数据仓库中的事务）、[Optimizing transactions for SQL Data Warehouse][]（优化 SQL 数据仓库的事务）

## 维护统计信息
不同于 SQL Server（将自动检测列并创建或更新收益列的统计数据），SQL 数据仓库需要手动维护统计信息。我们计划在将来改进这一点，但现在仍需要维护统计信息，以确保 SQL 数据仓库的计划优化。优化工具创建的计划只能使用可用的统计信息。**创建每个列的模板统计信息是开始使用统计信息的简单方式。** 更新统计信息和对数据做重大更改一样重要。保守的做法是每天或每次加载之后更新统计信息。创建和更新统计信息的性能与成本之间总有一些取舍。如果你发现维护所有统计信息所需时间太长，可能要更谨慎选择哪些列要进行统计信息、哪些列需要频繁更新。例如，可能想要更新每天都要添加新值的日期列。**对涉及联接的列、WHERE 子句中使用的列、在 GROUP BY 中找到的列进行统计信息，可以获得最大效益。**

另请参阅 [Manage statistics in SQL Data Warehouse][]（管理 SQL 数据仓库中的统计信息）、[CREATE STATISTICS (Transact-SQL)][]、[UPDATE STATISTICS (Transact-SQL)][]

## 将 INSERT 语句分组为批
使用 INSERT 语句一次性载入小型表、甚至定期重新加载查找，可能与使用类似于 `INSERT INTO MyLookup VALUES (1, 'Type 1')` 的语句一样正常执行。但是，如果一整天都要加载数千或数百万个行，你可能发现单个 INSERT 跟不上要求。请开发自己的进程将它们写入文件，以及另一个进程定期处理并加载此文件。

另请参阅 [Insert (Transact SQL)][]
 
## 使用 PolyBase 快速加载和导出数据
SQL 数据仓库支持通过多种工具（包括 Azure 数据工厂、PolyBase、BCP）来加载和导出数据。对于少量的数据，性能不是那么重要，任何工具都可以满足需求。但是，当要加载或导出大量数据，或者需要快速的性能时，PolyBase 是最佳选择。PolyBase 使用 SQL 数据仓库的 MPP（大规模并行处理）体系结构，因此加载和导出巨量数据的速度比其他任何工具更快。可使用 CTAS 或 INSERT INTO 来运行 PolyBase 加载。**使用 CTAS 可以减少事务日志记录，是加载数据最快的方法。** Azure 数据工厂也支持 PolyBase 加载。PolyBase 支持各种不同的文件格式，包括 Gzip 文件。**若要在使用 gzip 文本文件时获得最大的吞吐量，请将文件分成 60 个以上的文件让加载有最大化的并行度。** 若要更快的总吞吐量，请考虑并行加载数据。

另请参阅 [Load data into SQL Data Warehouse][]（将数据载入 SQL 数据仓库）、[Guide for using PolyBase in SQL Data Warehouse][]（在 SQL 数据仓库中使用 PolyBase 的指南）、[Azure SQL Data Warehouse loading patterns and strategies][]（Azure SQL 数据仓库加载模式和策略）、[CREATE EXTERNAL FILE FORMAT (Transact-SQL)][]、[Create Table As Select (CTAS) in SQL Data Warehouse][]（SQL 数据仓库中的 Create Table As Select (CTAS)）

## 哈希分布大型表
默认情况下，表是以轮循机制分布的。这可让用户更容易开始创建表，而不必确定应该如何分布其表。轮循机制表在某些工作负荷中执行良好，但通常选择分布列的执行性能将更好。按列分布表的性能远远高于轮循机制表的最常见例子是联接两个大型事实表。例如，如果有一个依 order\_id 分布的订单表，以及一个也是依 order\_id 分布的事务表，如果将订单数据联接到事务表上的 order\_id，此查询将变成传递查询，也就是数据移动操作将被消除。更少的步骤代表更快的查询。更少的数据移动也将让查询更快。这种解释只是大致的梗概。加载分布的表时，请确保传入数据的分布键没有排序，因为这将拖慢加载速度。有关选择分布列如何能提升性能，以及如何在 CREATE TABLES 语句的 WITH 子句中定义分布表的详细信息，请参阅以下链接。

另请参阅 [Hash distribution and its effect on query performance in SQL Data Warehouse][]（哈希分布及其对 SQL 数据仓库中查询性能的影响）、[Choosing hash distributed table vs. round-robin distributed table][]（选择哈希分布表与轮循机制分布表）、[CREATE TABLE (Azure SQL Data Warehouse, Parallel Data Warehouse)][]（CREATE TABLE（Azure SQL 数据仓库，并行数据仓库））、[CREATE TABLE AS SELECT (Azure SQL Data Warehouse)][]（CREATE TABLE AS SELECT（Azure SQL 数据仓库））

## 不要过度分区
尽管数据分区可以让数据维护变得有效率（通过分区切换或优化扫描将分区消除），太多的分区将让查询变慢。通常在 SQL Server 上运行良好的高数据粒度分区策略可能无法在 SQL 数据仓库上正常工作。如果每个分区的行数少于 1 百万，太多分区还将会降低聚集列存储索引的效率。请记住，SQL 数据仓库数据在幕后将数据分区成 60 个数据库，因此如果创建包含 100 个分区的表，实际上将导致 6000 个分区。每个工作负荷都不同，因此最佳建议是尝试不同的分区，找出最适合工作负荷的分区。请考虑比 SQL Server 上运行良好的数据粒度更低的粒度。例如，考虑使用每周或每月分区，而不是每日分区。

另请参阅 [Table partitions in SQL Data Warehouse][]（SQL 数据仓库中的表分区）

## 最小化事务大小
在事务中运行的 INSERT、UPDATE、DELETE 语句，失败时必须回滚。为了将长时间回滚的可能性降到最低，请尽可能将事务大小最小化。这可以通过将 INSERT、UPDATE、DELETE 语句分成小部分来达成。例如，如果预期 INSERT 需要 1 小时，可能的话，将 INSERT 分成 4 个部分，每个运行 15 分钟。使用特殊的最低限度日志记录方案，像是 CTAS、TRUNCATE、DROP TABLE 或 INSERT 空表，来降低回滚的风险。另一个消除回滚的作法是使用“仅元数据”操作（像是分区切换）进行数据管理。例如，不要运行 DELETE 语句来删除表中所有 order\_date 为 2001 年 10 月的行，而是将数据每月分区后，再从另一个表将有空分区之数据的分区调动出来（请参阅 ALTER TABLE 示例）。针对未分区的表，请考虑使用 CTAS 将想要保留的数据写入表中，而不是使用 DELETE。如果 CTAS 需要的时间一样长，则较安全的操作，是在它具有极小事务记录的条件下运行它，且必要时可以快速地取消。

另请参阅 [Transactions in SQL Data Warehouse][]（SQL 数据仓库中的事务）、[Optimizing transactions for SQL Data Warehouse][]（优化 SQL 数据仓库的事务）、[Table partitions in SQL Data Warehouse][]（SQL 数据仓库中的表分区）、[TRUNCATE TABLE (Transact-SQL)][]、[ALTER TABLE (Transact-SQL)][]、[Create Table As Select (CTAS) in SQL Data Warehouse][]（SQL 数据仓库中的 Create Table As Select (CTAS)）

## 使用最小可能的列大小
在定义 DDL 时，使用可支持数据的最小数据类型，将能够改善查询性能。这对 CHAR 和 VARCHAR 列尤其重要。如果列中最长的值是 25 个字符，请将列定义为 VARCHAR(25)。避免将所有字符列定义为较大的默认长度。此外，将列定义为 VARCHAR（当它只需要这样的大小时）而非 NVARCHAR。

另请参阅 [CREATE TABLE (Azure SQL Data Warehouse, Parallel Data Warehouse)][]（CREATE TABLE（Azure SQL 数据仓库，并行数据仓库））

## 对暂时性数据使用临时堆表
当在 SQL 数据仓库上暂时登录数据时，可能发现使用堆表可让整个过程更快速。如果加载数据只是在做运行更多转换之前的预备，将表载入堆表将会远快于将数据载入聚集列存储表。此外，将数据载入临时表也比将表载入永久存储更快速。临时表以“#”开头，且只有创建它的会话才能访问它，因此在某些情况下可能无法运行。堆表在 CREATE TABLE 的 WITH 子句中定义。如果你使用临时表，请记得同时在该临时表上创建统计信息。

另请参阅 [Temporary tables in SQL Data Warehouse][]（SQL 数据仓库中的临时表）、[CREATE TABLE (Azure SQL Data Warehouse, Parallel Data Warehouse)][]（CREATE TABLE（Azure SQL 数据仓库，并行数据仓库））、[CREATE TABLE AS SELECT (Azure SQL Data Warehouse)][]（CREATE TABLE AS SELECT（Azure SQL 数据仓库））

## 优化聚集列存储表
聚集列存储索引是将数据存储在 Azure SQL 数据仓库中最有效率的方式之一。默认情况下，SQL 数据仓库中的表将创建为聚集列存储。为了让列存储表的查询获得最佳性能，良好的分段质量很重要。当行在内存不足的状态下写入列存储表时，列存储分段质量可能降低。压缩行组中的行数可以测量分段质量。有关检测和改善聚集列存储表分段质量的逐步说明，请参阅[Troubleshooting][]（故障排除）中的 **Clustered Columnstore Segment Quality**（聚集列存储分段质量）部分。因为获取高质量的列存储分段相当重要，创建只用于加载中型或大型资源类别的特殊用户 ID 通常是个不错的主意。使用的 DWU 越小，要分配给加载用户的资源类越大。

由于列存储表通常要等到每个表有超过 1 百万个行之后才将会数据推送到压缩的列存储区段，并且每个 SQL 数据仓库表分区成 60 个表，根据经验法则，列存储表对于查询没有好处，除非表有超过 6 千万行。小于 6 千万列的表使用列存储索引似乎不太合理，但也无伤大雅。此外，如果将分区，则要考虑的是每个分区必须有 1 百万个行，使用聚集列存储索引才有益。如果表有 100 个分区，则它至少必须拥有 60 亿个行才将受益于聚集列存储（60 个分布区 * 100 个分区 * 1 百万行）。如果表在本示例中并没有 60 亿个行，请减少分区数目，或考虑改用堆表。使用辅助索引配合堆表而不是列存储表，也可能是值得进行的实验，看看是否可以获得较佳的性能。列存储表尚不支持辅助索引。

查询列存储表时，如果只选择需要的列，查询运行将更快速。

另请参阅 [Troubleshooting][]（故障排除）、[Manage columnstore indexes in Azure SQL Data Warehouse][]（管理 Azure SQL 数据仓库中的列存储索引）和 [Columnstore Indexes Guide][]（列存储索引指南）

## 使用较大的资源类来改善查询性能
SQL 数据仓库使用资源组作为将内存分配给查询的一种方式。默认情况下，所有用户都分配有小型资源类，此类授予每个分布区 100 MB 内存。因为永远将有 60 个分布区，每个分布区有至少 100 MB，整个系统的总内存分配为 6000 MB 或者刚好接近6 GB。有些查询，例如大型联接或载入聚集列存储表，将受益于较大的内存分配。某些查询，例如纯扫描，则不会获得任何好处。另一方面，使用较大的资源类会影响并行访问，因此将所有的用户移到大型资源类之前，要先将这一点纳入考虑。
 
另请参阅 [Concurrency and workload management in SQL Data Warehouse][]（SQL 数据仓库中的并发性和工作负荷管理）

## 使用较小的资源类来增加并发性
如果注意到用户查询似乎长时间延迟，可能是用户在较大资源类中运行，占用大量的并发性位置，而导致其他查询排入队列。若要确认用户的查询是否被排入队列，请运行 `SELECT * FROM sys.dm_pdw_waits` 看看是否返回任何行。

另请参阅 [Concurrency and workload management in SQL Data Warehouse][]（SQL 数据仓库中的并发性和工作负荷管理）、[sys.dm\_pdw\_waits (Transact-SQL)][]

## 使用 DMV 来监视和优化查询
SQL 数据仓库有多个 DMV 可用于监视查询执行。以下监视相关文章逐步说明了如何查看正在执行的查询的详细信息。若要在这些 DMV 中快速找到查询，可在查询中使用 LABEL 选项。

另请参阅 [Monitor your workload using DMVs][]（使用 DMV 监视工作负荷）、[Use labels to instrument queries in SQL Data Warehouse][]（使用标签检测 SQL 数据仓库中的查询）、[OPTION Clause (Transact-SQL)][]（OPTION 子句 (Transact-SQL)）、[sys.dm\_exec\_sessions (Transact-SQL)][]、[sys.dm\_pdw\_exec\_requests (Transact-SQL)][]、[sys.dm\_pdw\_request\_steps (Transact-SQL)][]、[sys.dm\_pdw\_sql\_requests (Transact-SQL)][]、[sys.dm\_pdw\_dms\_workers (Transact-SQL)]、[DBCC PDW\_SHOWEXECUTIONPLAN (Transact-SQL)][]、[sys.dm\_pdw\_waits (Transact-SQL)][]

## 其他资源
有许多地方能够找到关于如何使用 Azure SQL 数据仓库的信息。本文是 Azure 文档的一部分，有许多其他 Azure 文章和 MSDN 文章的链接。我们将观察读者对这些文章的反馈，并经常更新。如果认为文章很有帮助，请回答此问题：“此页面有帮助吗？”。不管答案是“是”还是“否”，都可以提供意见。如果发现文章很有帮助，但有任何意见，请单击“是”，并添加认为我们可以如何改善文章的任何意见。如果此问题没有出现，在每个 Azure 文章的最后一定可以找到；对于 MSDN 文章，请使用每个 MSDN 页右上角的“有任何建议吗？”链接。我们将非常重视你的反馈，并将对大多数反应采取操作。

如果你有 SQL 数据仓库方面的**功能建议**，请使用 [Azure SQL 数据仓库反馈][]页。添加你的请求或对其他请求投票可帮助我们确定功能的优先级。

我们还创建了 [Azure SQL 数据仓库 MSDN 论坛][]，你可以向其他用户和 SQL 数据仓库的产品小组提出问题。我们将主动观察此论坛，以确保你的问题获得其他用户或我们的回答。如果你想要提出有关 Stack Overflow 的问题，请访问 [Azure SQL 数据仓库 Stack Overflow 论坛][]。

<!--Image references-->

<!--Article references-->
[create a support ticket]: /documentation/articles/sql-data-warehouse-get-started-create-support-ticket/
[Concurrency and workload management in SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-develop-concurrency/
[Create Table As Select (CTAS) in SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-develop-ctas/
[Guide for using PolyBase in SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-load-polybase-guide/
[Hash distribution and its effect on query performance in SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-develop-hash-distribution-key/
[Load data into SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-overview-load/
[Load data with Azure Data Factory]: /documentation/articles/sql-data-warehouse-get-started-load-with-azure-data-factory/
[Load data with bcp]: /documentation/articles/sql-data-warehouse-load-with-bcp/
[Load data with PolyBase in SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-get-started-load-with-polybase/
[Manage columnstore indexes in Azure SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-manage-columnstore-indexes/
[Manage statistics in SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-develop-statistics/
[Monitor your workload using DMVs]: /documentation/articles/sql-data-warehouse-manage-monitor/
[Move data to and from Azure SQL Data Warehouse using Azure Data Factory]: /documentation/articles/data-factory-azure-sql-data-warehouse-connector/
[Optimizing transactions for SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-develop-best-practices-transactions/
[Pause compute resources]: /documentation/articles/sql-data-warehouse-manage-compute-overview/#pause-compute-bk
[Resume compute resources]: /documentation/articles/sql-data-warehouse-manage-compute-overview/#resume-compute-bk
[Scale compute resources]: /documentation/articles/sql-data-warehouse-manage-compute-overview/#scale-performance-bk
[Table design in SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-develop-table-design/
[Table partitions in SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-develop-table-partitions/
[Temporary tables in SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-develop-temporary-tables/
[Transactions in SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-develop-transactions/
[Troubleshooting]: /documentation/articles/sql-data-warehouse-troubleshoot/
[Use labels to instrument queries in SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-develop-label/

<!--MSDN references-->
[ALTER TABLE (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/ms190273.aspx
[Columnstore Indexes Guide]: https://msdn.microsoft.com/zh-cn/library/gg492088.aspx
[CREATE TABLE (Azure SQL Data Warehouse, Parallel Data Warehouse)]: https://msdn.microsoft.com/zh-cn/library/mt203953.aspx
[CREATE EXTERNAL FILE FORMAT (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/dn935026.aspx
[CREATE TABLE AS SELECT (Azure SQL Data Warehouse)]: https://msdn.microsoft.com/zh-cn/library/mt204041.aspx
[CREATE STATISTICS (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/ms188038.aspx
[DBCC PDW\_SHOWEXECUTIONPLAN (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/mt204017.aspx
[Insert (Transact SQL)]: https://msdn.microsoft.com/zh-cn/library/ms174335.aspx
[OPTION Clause (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/ms190322.aspx
[sys.dm\_exec\_sessions (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/ms176013.aspx
[sys.dm\_pdw\_exec\_requests (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/mt203887.aspx
[sys.dm\_pdw\_request\_steps (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/mt203913.aspx
[sys.dm\_pdw\_sql\_requests (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/mt203889.aspx
[sys.dm\_pdw\_dms\_workers (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/mt203878.aspx
[sys.dm\_pdw\_waits (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/mt203893.aspx
[TRUNCATE TABLE (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/ms177570.aspx
[UPDATE STATISTICS (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/ms187348.aspx

<!--Other Web references-->
[Choosing hash distributed table vs. round-robin distributed table]: https://blogs.msdn.microsoft.com/sqlcat/2015/08/11/choosing-hash-distributed-table-vs-round-robin-distributed-table-in-azure-sql-dw-service/
[Azure SQL 数据仓库反馈]: https://feedback.azure.com/forums/307516-sql-data-warehouse
[Azure SQL 数据仓库 MSDN 论坛]: https://social.msdn.microsoft.com/Forums/sqlserver/home?forum=AzureSQLDataWarehouse
[Azure SQL 数据仓库 Stack Overflow 论坛]: http://stackoverflow.com/questions/tagged/azure-sqldw
[Azure SQL Data Warehouse loading patterns and strategies]: https://blogs.msdn.microsoft.com/sqlcat/2016/02/06/azure-sql-data-warehouse-loading-patterns-and-strategies

<!---HONumber=Mooncake_0523_2016-->