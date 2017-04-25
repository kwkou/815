<properties
    pageTitle="管理 Azure SQL 数据仓库中的计算能力（概述）| Azure"
    description="Azure SQL 数据仓库中的性能横向扩展功能。 通过调整 DWU 数目进行横向扩展，或者通过暂停和恢复计算资源来节省成本。"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="barbkess"
    manager="jhubbard"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="e13a82b0-abfe-429f-ac3c-f2b6789a70c6"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.date="10/31/2016"
    wacn.date="04/24/2017"
    ms.author="barbkess"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="f66d435c0ce1e533ebd088935b02376ae6d09e39"
    ms.lasthandoff="04/14/2017" />

# <a name="manage-compute-power-in-azure-sql-data-warehouse-overview"></a>管理 Azure SQL 数据仓库中的计算能力（概述）
> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-data-warehouse-manage-compute-overview/)
- [门户](/documentation/articles/sql-data-warehouse-manage-compute-portal/)
- [PowerShell](/documentation/articles/sql-data-warehouse-manage-compute-powershell/)
- [REST](/documentation/articles/sql-data-warehouse-manage-compute-rest-api/)
- [TSQL](/documentation/articles/sql-data-warehouse-manage-compute-tsql/)

SQL 数据仓库的体系结构对存储和计算功能进行了分隔，允许每项功能单独进行缩放。 因此，你可以在扩大性能范围的同时节省成本，只需根据需要支付相关性能费用。

本概述文章介绍 SQL 数据仓库的以下性能扩展功能，并提供了这些功能的使用方式和时机方面的建议。

* 通过调整[数据仓库单位 (DWU)][data warehouse units (DWUs)] 对计算能力进行缩放
* 暂停或恢复计算资源

<a name="scale-performance-bk"></a>
##<a name="scale-performance"></a> 缩放性能
在 SQL 数据仓库中，你可以快速地进行性能缩放，只需增加或减少 CPU 计算资源、内存和 I/O 带宽即可。 若要对性能进行缩放，只需调整 SQL 数据仓库分配给你的数据库的[数据仓库单位 (DWU)][data warehouse units (DWUs)] 数即可。 SQL 数据仓库可以快速地进行更改，并处理针对硬件或软件的所有基础更改。

此外，你再也不需要为了获得优异的数据仓库性能，而研究处理器类型、需要的内存容量或存储空间类型。 如果将数据仓库放在云中，你就不再需要应付低级硬件问题。 相反，SQL 数据仓库会问你：你想要多快的数据处理速度？

### <a name="how-do-i-scale-performance"></a>如何进行性能缩放？
若要对计算能力进行弹性增减，只需对数据库的[数据仓库单位 (DWU)][data warehouse units (DWUs)] 设置进行更改即可。 随着添加更多的 DWU，性能将呈线性增加。  在较高的 DWU 级别，若要显著改善性能，需添加 100 以上的 DWU。 为了方便你选择有意义的 DWU 增长数，我们提供了会给出最佳结果的 DWU 级别。

若要调整 DWU，可以使用以下任何单个方法。

* [通过 Azure 门户缩放计算能力][Scale compute power with Azure portal]
* [通过 PowerShell 缩放计算能力][Scale compute power with PowerShell]
* [通过 REST API 缩放计算能力][Scale compute power with REST APIs]
* [通过 TSQL 缩放计算能力][Scale compute power with TSQL]

### <a name="how-many-dwus-should-i-use"></a>应该使用多少 DWU？
SQL 数据仓库的性能以线性方式缩放，在几秒内就能从某个计算比例更改为另一个（例如从 100 个 DWU 到 2000 个 DWU）。 这样可以让你灵活地对不同的 DWU 设置进行试验，直至确定适合你方案的最佳选择。

若要了解理想的 DWU 值，请尝试在加载数据之后增加和减少 DWU 并运行几个查询。 由于缩放很快就能完成，你可以在一个小时或更少时间内尝试一些不同级别的性能。 请务必记住，SQL 数据仓库设计用于处理大量数据，若要了解其处理大数据的真正能力，尤其是我们提供的较大规模数据，你需要使用接近或超过 1 TB 的大型数据集。

针对工作负荷查找最佳 DWU 时，建议你：

1. 对于正在开发的数据仓库，可以从少量的 DWU 开始。  一个好的起点为 DW400 或 DW200。
2. 监视应用程序性能，将所选 DWU 数目与观测到的性能变化进行比较。
3. 通过假设线性缩放，判断需要将性能加快或减慢多少才能达到要求的最佳性能级别。
4. 如果你要加快或减慢工作负荷的执行速度，则可按比例相应地提高或降低 DWU 数目。 服务将快速响应并调整计算资源，以符合新的 DWU 要求。
5. 继续进行调整，直到达到业务要求的最佳性能级别。

### <a name="when-should-i-scale-dwus"></a>何时应进行 DWU 缩放？
当你需要更快的结果时，可以增加 DWU 并为较高的性能付费。  当你需要较小的计算能力时，可以减少 DWU 并且仅针对需要的项目付费。

关于何时进行 DWU 缩放的建议：

1. 如果应用程序的工作负荷持续变动，请将 DWU 级别上调或下调，以配合高峰和离峰点。 比方说，如果工作负荷高峰通常出现在月底，则可以规划在高峰期添加更多 DWU，然后在高峰期结束后减少。
2. 执行繁重的数据加载或转换操作时，增加 DWU 数目即可更快速地获取数据。

<a name="pause-compute-bk"></a>

## <a name="pause-compute"></a>暂停计算

[AZURE.INCLUDE [SQL Data Warehouse pause description](../../includes/sql-data-warehouse-pause-description.md)]

若要暂停数据库，请使用下列任意方法之一。

* [通过 Azure 门户暂停计算][Pause compute with Azure portal]
* [通过 PowerShell 暂停计算][Pause compute with PowerShell]
* [通过 REST API 暂停计算][Pause compute with REST APIs]

<a name="resume-compute-bk"></a>

## <a name="resume-compute"></a>恢复计算

[AZURE.INCLUDE [SQL Data Warehouse resume description](../../includes/sql-data-warehouse-resume-description.md)]

若要恢复数据库，请使用下列任意方法之一。

* [通过 Azure 门户恢复计算][Resume compute with Azure portal]
* [通过 PowerShell 恢复计算][Resume compute with PowerShell]
* [通过 REST API 恢复计算][Resume compute with REST APIs]

## <a name="permissions"></a>权限
缩放数据库将需要 [ALTER DATABASE][ALTER DATABASE]中所述的权限。  暂停和恢复将需要 [SQL DB 参与者][SQL DB Contributor] 权限，具体说来就是 Microsoft.Sql/servers/databases/action。

<a name="next-steps-bk"></a>

## <a name="next-steps"></a>后续步骤
请参阅以下文章，以帮助了解其他一些主要的性能概念：

* [工作负荷和并发管理][Workload and concurrency managment]
* [表设计概述][Table design overview]
* [表分布][Table distribution]
* [表索引][Table indexing]
* [表分区][Table partitioning]
* [表统计信息][Table statistics]
* [最佳实践][Best practices]

<!--Image reference-->

<!--Article references-->
[data warehouse units (DWUs)]: /documentation/articles/sql-data-warehouse-overview-what-is/

[Scale compute power with Azure portal]: /documentation/articles/sql-data-warehouse-manage-compute-portal/#scale-compute-power
[Scale compute power with PowerShell]: /documentation/articles/sql-data-warehouse-manage-compute-powershell/#scale-compute-bk
[Scale compute power with REST APIs]: /documentation/articles/sql-data-warehouse-manage-compute-rest-api/#scale-compute-bk
[Scale compute power with TSQL]: /documentation/articles/sql-data-warehouse-manage-compute-tsql/#scale-compute-bk

[capacity limits]: /documentation/articles/sql-data-warehouse-service-capacity-limits/

[Pause compute with Azure portal]: /documentation/articles/sql-data-warehouse-manage-compute-portal/#pause-compute-bk
[Pause compute with PowerShell]: /documentation/articles/sql-data-warehouse-manage-compute-powershell/#pause-compute-bk
[Pause compute with REST APIs]: /documentation/articles/sql-data-warehouse-manage-compute-rest-api/#pause-compute-bk

[Resume compute with Azure portal]: /documentation/articles/sql-data-warehouse-manage-compute-portal/#resume-compute-bk
[Resume compute with PowerShell]: /documentation/articles/sql-data-warehouse-manage-compute-powershell/#resume-compute-bk
[Resume compute with REST APIs]: /documentation/articles/sql-data-warehouse-manage-compute-rest-api/#resume-compute-bk

[Workload and concurrency managment]: /documentation/articles/sql-data-warehouse-develop-concurrency/
[Table design overview]: /documentation/articles/sql-data-warehouse-tables-overview/
[Table distribution]: /documentation/articles/sql-data-warehouse-tables-distribute/
[Table indexing]: /documentation/articles/sql-data-warehouse-tables-index/
[Table partitioning]: /documentation/articles/sql-data-warehouse-tables-partition/
[Table statistics]: /documentation/articles/sql-data-warehouse-tables-statistics/
[Best practices]: /documentation/articles/sql-data-warehouse-best-practices/
[development overview]: /documentation/articles/sql-data-warehouse-overview-develop/

[SQL DB Contributor]: /documentation/articles/role-based-access-built-in-roles/#sql-db-contributor

<!--MSDN references-->
[ALTER DATABASE]: https://msdn.microsoft.com/zh-cn/library/mt204042.aspx

<!--Other Web references-->
[Azure portal]: http://portal.azure.cn/

<!--Update_Description:update meta properties,wording update; update reference-->