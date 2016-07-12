<!-- manage-portal only in ibiza portal -->
<properties
   pageTitle="管理 Azure SQL 数据仓库中的计算能力（概述）| Azure"
   description="Azure SQL 数据仓库中的性能横向扩展功能。通过调整 DWU 数目进行横向扩展，或者通过暂停和恢复计算资源来节省成本。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="barbkess"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/06/2016"
   wacn.date="06/20/2016"/>

# 管理 Azure SQL 数据仓库中的计算能力（概述）

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-data-warehouse-overview-manage-compute/)
- [PowerShell](/documentation/articles/sql-data-warehouse-manage-compute-powershell/)
- [REST](/documentation/articles/sql-data-warehouse-manage-compute-rest-api/)
- [TSQL](/documentation/articles/sql-data-warehouse-manage-compute-tsql/)

SQL 数据仓库的体系结构对存储和计算功能进行了分隔，允许每项功能单独进行缩放。因此，你可以在扩大性能范围的同时节省成本，只需根据需要支付相关性能费用。

本概述文章介绍 SQL 数据仓库的以下性能扩展功能，并提供了这些功能的使用方式和时机方面的建议。

- 通过调整数据仓库单位 (DWU) 对计算能力进行缩放
- 暂停或恢复计算资源

<a name="scale-performance-bk"></a>

## 缩放性能

在 SQL 数据仓库中，你可以快速地进行性能缩放，只需增加或减少 CPU 计算资源、内存和 I/O 带宽即可。进行性能缩放时，只需调整 SQL 数据仓库分配给你的数据库的数据仓库单位 (DWU) 数即可。SQL 数据仓库可以快速地进行更改，并处理针对硬件或软件的所有基础更改。

>[AZURE.NOTE] 此外，你再也不需要为了获得优异的数据仓库性能，而研究处理器类型、需要的内存容量或存储空间类型。如果将数据仓库放在云中，你就不再需要应付低级硬件问题。相反，SQL 数据仓库会问你：你想要多快的数据处理速度？

### 什么是 DWU？

数据仓库单位 (DWU) 是一种度量单位，可以在任何特定的时间衡量数据库的基础计算功能。当 DWU 数目增加时，SQL 数据仓库操作（例如数据加载或查询）可跨越更多分布式 CPU 和内存资源并行运行。这可减少延迟并改善性能。

DWU 基于加载速率和扫描速率。增加 DWU 时，也会增加加载速率和扫描速率。

- **加载速率**。SQL 数据仓库每秒可以引入的记录数。具体说来，这是指 SQL 数据仓库可以通过 PolyBase 从 Azure Blob 存储导入聚集列存储索引的记录数。 

- **扫描速率**。查询每秒可以从 SQL 数据仓库检索的记录数。具体说来，这是指 SQL 数据仓库在针对聚集列存储索引运行数据仓库查询时可以返回的记录数。为了测试扫描速率，我们使用标准数据仓库查询来扫描大量的行，然后执行复杂聚合。这是一种 IO 和 CPU 密集型操作。

>[AZURE.NOTE] 我们正在度量和开发一些重要的性能增强功能，不久就能给大家带来预期的速率。在预览期，我们将持续增强服务质量（例如提高压缩和缓存）以提高这些速率，并确保能够以可预测的方式调整这些速率。

如需 DWU 的列表，请参阅[容量限制][]这篇文章中的“服务级别目标”。

### 如何进行性能缩放？

若要对计算能力进行弹性增减，只需对数据库的数据仓库单位 (DWU) 设置进行更改即可。SQL 数据仓库在后台通过 SQL 数据库的快速而简单的部署功能来更改 CPU 和内存分配。

DWU 以 100 个块为单位进行分配，但并非所有块都可用。DWU 增加时，性能呈线性增加。在较高的 DWU 级别，若要显著改善性能，需添加 100 以上的 DWU。为了方便你选择有意义的 DWU 增长数，我们提供了会给出最佳结果的 DWU 级别。
 
若要调整 DWU，可以使用以下任何单个方法。

- [通过 PowerShell 缩放计算能力][]
- [通过 REST API 缩放计算能力][]
- [通过 TSQL 缩放计算能力][]

### 应该使用多少 DWU？
 
SQL 数据仓库的性能以线性方式缩放，在几秒内就能从某个计算比例更改为另一个（例如从 100 个 DWU 到 2000 个 DWU）。这样可以让你灵活地对不同的 DWU 设置进行试验，直至确定适合你方案的最佳选择。

> [AZURE.NOTE] 在数据量较小时，可能看不到预期的性能缩放效果。我们建议从大于或等于 1 TB 的数据卷开始，以便获得准确的性能测试结果。

针对工作负荷查找最佳 DWU 时，建议你：

1. 对于正在开发的数据仓库，可以从少量的 DWU 开始。
2. 监视应用程序性能，将所选 DWU 数目与观测到的性能变化进行比较。
3. 通过假设线性缩放，判断需要将性能加快或减慢多少才能达到要求的最佳性能级别。
4. 如果你要加快或减慢工作负荷的执行速度，则可按比例相应地提高或降低 DWU 数目。服务将快速响应并调整计算资源，以符合新的 DWU 要求。
5. 继续进行调整，直到达到业务要求的最佳性能级别。

### 何时应进行 DWU 缩放？

总体而言，我们希望 DWU 尽可能简单。当你需要更快的结果时，可以增加 DWU 并为较高的性能付费。当你需要较小的计算能力时，可以减少 DWU 并且仅针对需要的项目付费。

关于何时进行 DWU 缩放的建议：

1. 如果应用程序的工作负荷持续变动，请将 DWU 级别上调或下调，以配合高峰和离峰点。比方说，如果工作负荷高峰通常出现在月底，则可以规划在高峰期添加更多 DWU，然后在高峰期结束后减少。
1. 执行繁重的数据加载或转换操作时，增加 DWU 数目即可更快速地获取数据。

<a name="pause-compute-bk"></a>

## 暂停计算

[AZURE.INCLUDE [SQL Data Warehouse pause description（SQL 数据仓库暂停说明）](../includes/sql-data-warehouse-pause-description)]

若要暂停数据库，请使用下列任意方法之一。

- [通过 PowerShell 暂停计算][]
- [通过 REST API 暂停计算][]

<a name="resume-compute-bk"></a>

## 恢复计算

[AZURE.INCLUDE [SQL Data Warehouse resume description（SQL 数据仓库恢复说明）](../includes/sql-data-warehouse-resume-description)]

若要恢复数据库，请使用下列任意方法之一。

- [通过 PowerShell 恢复计算][]
- [通过 REST API 恢复计算][]

<a name="next-steps-bk"></a>

## 后续步骤
请参阅以下文章，以帮助了解其他一些关键的性能与缩放性概念：

- [并发模型][]
- [设计表][]
- [为表选择哈希分布键][]
- [用于改善性能的统计信息][]

<!--Image reference-->

<!--Article references-->

[通过 PowerShell 缩放计算能力]: /documentation/articles/sql-data-warehouse-manage-compute-powershell/#task-1-scale-performance
[通过 REST API 缩放计算能力]: /documentation/articles/sql-data-warehouse-manage-compute-rest-api/#task-1-scale-performance
[通过 TSQL 缩放计算能力]: /documentation/articles/sql-data-warehouse-manage-compute-tsql/

[容量限制]: /documentation/articles/sql-data-warehouse-service-capacity-limits/

[通过 PowerShell 暂停计算]: /documentation/articles/sql-data-warehouse-manage-compute-powershell/#scale-compute-bk
[通过 REST API 暂停计算]: /documentation/articles/sql-data-warehouse-manage-compute-rest-api/#scale-compute-bk
[通过 PowerShell 恢复计算]: /documentation/articles/sql-data-warehouse-manage-compute-powershell/#resume-compute-bk
[通过 REST API 恢复计算]: /documentation/articles/sql-data-warehouse-manage-compute-rest-api/#resume-compute-bk

[并发模型]: /documentation/articles/sql-data-warehouse-develop-concurrency/
[设计表]: /documentation/articles/sql-data-warehouse-develop-table-design/
[为表选择哈希分布键]: /documentation/articles/sql-data-warehouse-develop-hash-distribution-key/
[用于改善性能的统计信息]: /documentation/articles/sql-data-warehouse-develop-statistics/
[development overview]: /documentation/articles/sql-data-warehouse-overview-develop/



<!--MSDN references-->


<!--Other Web references-->

[Azure portal]: http://manage.windowsazure.cn/

<!---HONumber=Mooncake_0613_2016-->