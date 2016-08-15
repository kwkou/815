<properties
    pageTitle="Azure SQL 数据库弹性数据库查询概述 | Azure"
    description="弹性查询功能概述"    
    services="sql-database"
    documentationCenter=""  
    manager="jhubbard"
    authors="sidneyh"/>

<tags
    ms.service="sql-database"
    ms.date="04/04/2016"
    wacn.date="06/14/2016" />

# Azure SQL 数据库弹性数据库查询概述（预览版）

使用弹性数据库查询功能（处于预览状态）可以跨 Azure SQL 数据库 (SQLDB) 中的多个数据库运行 Transact-SQL 查询。它允许你执行跨数据库查询以访问远程表，以及连接 Microsoft 和第三方工具（Excel、PowerBI、Tableau 等）以跨多个数据库的数据层进行查询。使用此功能，可以将查询扩大到 SQL 数据库中的较大数据层，并直观显示商业智能 (BI) 报表中的结果。
若要开始构建弹性数据库查询应用程序，请参阅[弹性数据库查询入门](/documentation/articles/sql-database-elastic-query-getting-started/)。

## 弹性数据库查询的新增功能

* 现在可以在 T-SQL 中完全定义使用单个远程数据库的跨数据库查询方案。这允许对远程数据库进行只读查询。这为当前的本地 SQL Server 客户提供了一个选项，将使用由三部分和四部分组成的名称或链接服务器的应用程序迁移到 SQL 数据库。
* 除了高级性能层之外，现在标准性能层也支持弹性查询。有关较低性能层的性能限制，请参阅下面有关预览版限制的部分。
* 弹性查询现在可以将 SQL 参数推送到远程数据库执行。
* 使用 sp\_execute\_remote 的远程存储过程调用或远程函数调用现在可以使用类似于 [sp\_executesql](https://msdn.microsoft.com/zh-cn/library/ms188001.aspx) 的参数。
* 从远程数据库中检索大型结果集的性能已改进。
* 现在使用弹性查询的外部表可以引用具有不同架构或表名的远程表。

## 弹性数据库查询方案

目标是便于实现多个数据库参与生成单个总体结果中的行的查询方案。查询可以由用户或应用程序直接编写，也可以通过与数据库连接的工具间接编写。这在使用商业 BI 或数据集成工具创建报表或任何不能更改的应用程序时特别有用。使用弹性查询，你可以利用 Excel、PowerBI、Tableau 或 Cognos 等工具中熟悉的 SQL Server 连接体验跨多个数据库进行查询。
弹性查询可让你通过 SQL Server Management Studio 或 Visual Studio 发出的查询轻松访问整个数据库集合，并便于实现从实体框架或其他 ORM 环境跨数据库查询。图 1 显示了这样一个方案：其中使用[弹性数据库客户端库](/documentation/articles/sql-database-elastic-database-client-library/)的现有云应用程序在扩大的数据层的基础上构建，并且弹性查询用于跨数据库报告。

**图 1** 在扩大的数据层上使用的弹性数据库查询

![在扩大的数据层上使用的弹性数据库查询][1]

弹性查询的客户方案的特征包括以下拓扑：

* **垂直分区 - 跨数据库查询**（拓扑 1）：在数据层中的多个数据库之间对数据进行垂直分区。通常，不同的表集驻留在不同的数据库上。这意味着不同数据库上的架构是不同的。例如，清单的所有表都位于一个数据库上，而与会计相关的所有表都位于第二个数据库上。采用此拓扑的常见使用案例需要使用一个查询跨多个数据库中的表进行查询或编译报表。
* **水平分区 - 分片**（拓扑 2）：将数据进行水平分区以将行分布到扩大的数据层上。使用此方法时，所有参与数据库中的架构是相同的。此方法也称为“分片”。可以使用 (1) 弹性数据库客户端库或 (2) 自我分片来执行和管理分片。可使用弹性查询跨多个分片查询或编译报表。

> [AZURE.NOTE] 弹性数据库查询最适合于其中大部分处理可以在数据层上执行的临时报告方案。对于使用更复杂查询的大量报表工作负荷或数据仓库方案，还可以考虑使用 [Azure SQL 数据仓库](/home/features/sql-data-warehouse)。


## 弹性数据库查询拓扑

### 拓扑 1：垂直分区 - 跨数据库查询

若要开始编写代码，请参阅[跨数据库查询（垂直分区）入门](/documentation/articles/sql-database-elastic-query-getting-started-vertical/)。

弹性查询可用于使位于 SQLDB 数据库中的数据可供其他 SQLDB 数据库使用。这允许从一个数据库的查询引用任何其他远程 SQLDB 数据库中的表。第一步是定义每个远程数据库的外部数据源。外部数据源在你要从中获取对远程数据库中的表的访问权限的本地数据库中定义。在远程数据库上不必进行任何更改。对于不同数据库具有不同架构的典型垂直分区方案，可以使用弹性查询来实现常见使用案例（例如，访问引用数据和跨数据库查询）。

**引用数据**：拓扑 1 用于引用数据管理。在下图中，包含引用数据的两个表（T1 和 T2）保存在专用数据库中。使用弹性查询，你现在可以从其他数据库远程访问表 T1 和 T2，如图中所示。如果引用表较小或对引用表的远程查询包含选择性谓词，请使用拓扑 1。

**图 2** 垂直分区 - 使用弹性查询来查询引用数据

![垂直分区 - 使用弹性查询来查询引用数据][3]

**跨数据库查询**：弹性查询支持需要跨多个 SQLDB 数据库进行查询的用例。图 3 显示了四个不同的数据库：CRM、库存、人力资源和产品。在其中一个数据库中执行的查询还需要访问一个或所有其他数据库。使用弹性查询时，你可以通过在这四个数据库的每个数据库上运行几个简单的 DDL 语句来为此案例配置数据库。经过此一次性配置后，对远程表的访问就像从 T-SQL 查询或 BI 工具引用本地表一样简单。如果远程查询不返回大型结果，建议使用此方法。

**图 3** 垂直分区 - 使用弹性查询来跨多个数据库查询

![垂直分区 - 使用弹性查询来跨多个数据库查询][4]

### 拓扑 2：水平分区 - 分片

若要开始编写代码，请参阅[弹性数据库查询入门 - 水平分区（分片）](/documentation/articles/sql-database-elastic-query-getting-started/)

使用弹性查询对分片（即，水平分区）的数据层执行报表任务需要[弹性数据库分片映射](/documentation/articles/sql-database-elastic-scale-shard-map-management/)来表示数据层的数据库。通常情况下，仅在此方案中使用单个分片映射，并且具有弹性查询功能的专用数据库将作为报表查询的入口点。只有此专用数据库需要访问分片映射。图 2 说明了此拓扑及其使用弹性查询数据库和分片映射的配置。请注意，只有弹性查询数据库需要是 Azure SQL 数据库 v12 数据库。数据层中的数据库可以是任何 Azure SQL 数据库版本。有关弹性数据库客户端库和创建分片映射的详细信息，请参阅[分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management/)。

**图 4** 水平分区 - 使用弹性查询实现分片数据层上的报告

![水平分区 - 使用弹性查询实现分片数据层上的报告][5]

> [AZURE.NOTE] 专用弹性数据库查询数据库必须是 SQL 数据库 v12 数据库。对分片本身没有任何限制。


## 实现弹性数据库查询

以下各节将讨论为垂直和水平分区方案实现弹性查询的步骤。它们还引用了有关不同分区方案的更详细文档。

### 垂直分区 - 跨数据库查询

以下步骤为需要访问远程 SQLDB 数据库上的表的垂直分区方案配置弹性数据库查询：

*    [CREATE MASTER KEY](https://msdn.microsoft.com/zh-cn/library/ms174382.aspx) mymasterkey
*    [CREATE DATABASE SCOPED CREDENTIAL](https://msdn.microsoft.com/zh-cn/library/mt270260.aspx) mycredential
*    [CREATE/DROP EXTERNAL DATA SOURCE](https://msdn.microsoft.com/zh-cn/library/dn935022.aspx) mydatasource of type **RDBMS**
*    [CREATE/DROP EXTERNAL TABLE](https://msdn.microsoft.com/zh-cn/library/dn935021.aspx) mytable

运行 DDL 语句后，你可以访问远程表“mytable”就像它是本地表一样。Azure SQL 数据库将自动打开与远程数据库的连接，处理远程数据库上的请求并返回结果。
有关垂直分区方案所需的步骤的详细信息，可以在[垂直分区的弹性查询](/documentation/articles/sql-database-elastic-query-vertical-partitioning/)中找到。

### 水平分区 - 分片

以下步骤为需要访问（通常）位于多个远程 SQLDB 数据库上的一组表的水平分区方案配置弹性数据库查询：

*    [CREATE MASTER KEY](https://msdn.microsoft.com/zh-cn/library/ms174382.aspx) mymasterkey
*    [CREATE DATABASE SCOPED CREDENTIAL](https://msdn.microsoft.com/zh-cn/library/mt270260.aspx) mycredential
*    使用弹性数据库客户端库创建表示数据层的[分片映射](/documentation/articles/sql-database-elastic-scale-shard-map-management/)。   
*    [CREATE/DROP EXTERNAL DATA SOURCE](https://msdn.microsoft.com/zh-cn/library/dn935022.aspx) mydatasource of type **SHARD\_MAP\_MANAGER**
*    [CREATE/DROP EXTERNAL TABLE](https://msdn.microsoft.com/zh-cn/library/dn935021.aspx) mytable

执行这些步骤后，你便可以访问水平分区的表“mytable”就像它是本地表一样。Azure SQL 数据库将自动打开与远程数据库（在其中表以物理方式存储）的多个并行连接，处理远程数据库上的请求并返回结果。
有关水平分区方案所需的步骤的详细信息，可以在[水平分区的弹性查询](/documentation/articles/sql-database-elastic-query-horizontal-partitioning/)中找到。

## T-SQL 查询
你定义外部数据源和外部表后，便可以使用常规 SQL Server 连接字符串连接到你在其中定义了外部表的数据库。然后可以通过该连接对外部表运行 T-SQL 语句，但具有下述限制。你可以在[水平分区](/documentation/articles/sql-database-elastic-query-horizontal-partitioning/)和[垂直分区](/documentation/articles/sql-database-elastic-query-vertical-partitioning/)的文档主题中找到 T-SQL 查询的详细信息和示例。

## 工具的连接
可以使用常规 SQL Server 连接字符串将应用程序和 BI 或数据集成工具连接到具有外部表的数据库。请确保支持将 SQL Server 用作工具的数据源。连接后，可以引用弹性查询数据库和该数据库中的外部表，就像你会对使用工具连接的任何其他 SQL Server 数据库所做的一样。

## 成本

弹性查询包含在 Azure SQL 数据库的数据库成本中。请注意，支持远程数据库与弹性查询终结点在不同的数据中心的拓扑，但将按常规 [Azure 费率](/pricing/details/data-transfer)对远程数据库的数据流出量进行收费。

## 预览版限制
* 在标准性能层上运行第一个弹性查询可能需要长达几分钟时间。此时间是加载弹性查询功能所必需的；使用更高级性能层可提高加载性能。
* 尚不支持从 SSMS 或 SSDT 对外部数据源或外部表进行脚本编写。
* SQL 数据库的导入/导出尚不支持外部数据源和外部表。如果需要使用导入/导出，请在导出之前删除这些对象，然后在导入后重新创建它们。
* 弹性数据库查询当前仅支持对外部表进行只读访问。但是，你可以对在其中定义了外部表的数据库使用完整的 T-SQL 功能。这可能对以下操作很有用：例如，使用（例如）SELECT <column_list> INTO <local_table> 持久保存临时结果，或在引用外部表的弹性查询数据库中定义存储过程。
* 除了 nvarchar (max) 外，外部表定义也不支持 LOB 类型。一种解决方法是，可以在远程数据库上创建将 LOB 类型强制转换为 nvarchar(max) 的视图，通过该视图（而不是基表）定义外部表，然后在查询中将其强制转换回原始 LOB 类型。
* 当前不支持外部表上的列统计信息。支持表统计信息，但需要手动创建。

## 反馈
请在下面的 Disqus、MSDN 论坛或 Stackoverflow 上与我们共享有关你使用弹性查询的体验的反馈。我们对有关该服务的所有类型反馈（缺陷、未完善处、功能差距）感兴趣。

## 详细信息

你可以在以下文档中找到有关跨数据库查询和垂直分区方案的详细信息：

* [跨数据库查询和垂直分区概述](/documentation/articles/sql-database-elastic-query-vertical-partitioning/)
* 建议阅读可让完整的工作示例在几分钟内运行的分步教程：[跨数据库查询入门（垂直分区）](/documentation/articles/sql-database-elastic-query-getting-started-vertical/)。


有关水平分区和分片方案的详细信息可在此处找到：

* [水平分区和分片概述](/documentation/articles/sql-database-elastic-query-horizontal-partitioning/)
* 建议阅读可让完整的工作示例在几分钟内运行的分步教程：[弹性数据库查询入门 - 水平分区（分片）](/documentation/articles/sql-database-elastic-query-getting-started/)。


[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-query-overview/overview.png
[2]: ./media/sql-database-elastic-query-overview/topology1.png
[3]: ./media/sql-database-elastic-query-overview/vertpartrrefdata.png
[4]: ./media/sql-database-elastic-query-overview/verticalpartitioning.png
[5]: ./media/sql-database-elastic-query-overview/horizontalpartitioning.png

<!--anchors-->

<!---HONumber=Mooncake_0606_2016-->