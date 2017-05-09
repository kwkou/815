<properties
    pageTitle="管理 Azure SQL 数据仓库中的计算能力（概述）| Azure"
    description="Azure SQL 数据仓库中的性能横向扩展功能。 通过调整 DWU 数目进行横向扩展，或者通过暂停和恢复计算资源来节省成本。"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="hirokib"
    manager="johnmac"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="e13a82b0-abfe-429f-ac3c-f2b6789a70c6"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.custom="manage"
    ms.date="03/22/2017"
    wacn.date="05/08/2017"
    ms.author="elbutter"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="bddb370127988cc431c42eb2c2a158ff674b96d9"
    ms.lasthandoff="04/28/2017" />

# <a name="manage-compute-power-in-azure-sql-data-warehouse-overview"></a>管理 Azure SQL 数据仓库中的计算能力（概述）
> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-data-warehouse-manage-compute-overview/)
- [门户](/documentation/articles/sql-data-warehouse-manage-compute-portal/)
- [PowerShell](/documentation/articles/sql-data-warehouse-manage-compute-powershell/)
- [REST](/documentation/articles/sql-data-warehouse-manage-compute-rest-api/)
- [TSQL](/documentation/articles/sql-data-warehouse-manage-compute-tsql/)

SQL 数据仓库的体系结构对存储和计算功能进行了分隔，允许每项功能单独进行缩放。 因此，可以独立于数据量，根据性能需求缩放计算资源。 此体系结构的自然结果是，计算和存储的[计费][billed]是独立的。 

本概述文章介绍如何扩展 SQL 数据仓库，以及如何利用 SQL 数据仓库的暂停、恢复和缩放功能。 请查阅[数据仓库单位 (DWU)][data warehouse units (DWUs)]页，了解 DWU 与性能的关联方式。 

## <a name="how-compute-management-operations-work-in-sql-data-warehouse"></a>SQL 数据仓库中计算管理操作的工作原理
SQL 数据仓库的体系结构由控制节点、计算节点和跨 60 个分布区的存储层组成。 

在 SQL 数据仓库中，系统的头节点管理元数据，并且包含分布式的查询优化器中的正常活动会话。 此头节点下是计算节点和存储层。 对于 DWU 400，系统具有一个头节点、四个计算节点和存储层，包含 60 个分布区。 

进行缩放或暂停操作时，系统将首先将终止所有传入的查询，并会回滚事务以确保一致的状态。 为缩放操作，缩放仅后会进行此事务回滚已完成。 对于向上缩放操作，额外的所需数量的系统设置计算节点，并重新附加到的存储层的计算节点开始。 对于缩减操作，不需要的节点将发布和剩余的计算节点重新自身附加到相应数量的分发。 对于暂停操作，所有计算节点的发行和你的系统将进行各种元数据操作，以使最终系统处于稳定状态。

| DWU  | 计算节点的 \# | 每个节点的分布区 \# |
| ---- | ------------------ | ---------------------------- |
| 100  | 1                  | 60                           |
| 200  | 2                  | 30                           |
| 300  | 3                  | 20                           |
| 400  | 4                  | 15                           |
| 500  | 5                  | 12                           |
| 600  | 6                  | 10                           |
| 1000 | 10                 | 6                            |
| 1200 | 12                 | 5                            |
| 1500 | 15                 | 4                            |
| 2000 | 20                 | 3                            |
| 3000 | 30                 | 2                            |
| 6000 | 60                 | 1                            |

用于管理计算的三个主要功能是：

1. 暂停
2. 恢复
3. 缩放

其中每项操作可能需要几分钟才能完成。 如果自动将缩放/暂停/继续，可能想要实现逻辑，确保某些操作具有已完成，然后再继续进行另一项操作。 

检查通过不同的终结点的数据库状态将允许为了正确地实现这种操作的自动化。 门户将提供完成后通知运算和数据库的当前状态，但不允许对以编程方式检查状态。 

>  [AZURE.NOTE]
>
>  计算所有终结点之间不存在管理功能。
>
>  

|              | 暂停/恢复 | 缩放 | 检查数据库状态 |
| ------------ | ------------ | ----- | -------------------- |
| Azure 门户 | 是          | 是   | **否**               |
| PowerShell   | 是          | 是   | 是                  |
| REST API     | 是          | 是   | 是                  |
| T-SQL        | **否**       | 是   | 是                  |

## <a name="scale-compute-bk"></a><a name="scale-compute"></a>缩放计算

SQL 数据仓库中的性能以[数据仓库单位 (DWU)][data warehouse units (DWUs)] 度量，这是 CPU、内存和 I/O 带宽等计算资源的抽象度量值。 想要缩放其系统性能的用户可以通过不同的方式实现此目的，例如通过门户、T-SQL 和 REST API。 

### <a name="how-do-i-scale-compute"></a>如何缩放计算资源？
可通过更改 DWU 设置来管理 SQL 数据仓库的计算能力。 为特定的操作添加更多 DWU 后，性能会[线性][linearly]提高。  我们提供确保性能将更改显著时是向上或向下扩展系统的 DWU 产品。 

若要调整 DWU，可以使用以下任何单个方法。

* [通过 Azure 门户缩放计算能力][Scale compute power with Azure portal]
* [通过 PowerShell 缩放计算能力][Scale compute power with PowerShell]
* [通过 REST API 缩放计算能力][Scale compute power with REST APIs]
* [通过 TSQL 缩放计算能力][Scale compute power with TSQL]

### <a name="how-many-dwus-should-i-use"></a>应该使用多少 DWU？

若要了解理想的 DWU 值，请尝试在加载数据之后增加和减少 DWU 并运行几个查询。 由于缩放很快就能完成，可以在一个小时或更少时间内尝试一些不同级别的性能。 

> [AZURE.NOTE] 
> SQL 数据仓库旨在处理大量数据。 若要了解数据仓库的真正缩放功能，尤其是大规模 DWU 处理能力，可以使用接近或超过 1 TB 的大型数据集。

针对工作负荷查找最佳 DWU 时，建议你：

1. 对于正在开发的数据仓库，可以从较小的 DWU 性能级别开始。  一个好的起点为 DW400 或 DW200。
2. 监视应用程序性能，将所选 DWU 数目与观测到的性能变化进行比较。
3. 通过假设线性缩放，判断需要将性能加快或减慢多少才能达到要求的最佳性能级别。
4. 如果你要加快或减慢工作负荷的执行速度，则可按比例相应地提高或降低 DWU 数目。 
5. 继续进行调整，直到达到业务要求的最佳性能级别。

> [AZURE.NOTE]
>
> 此外，如果可以计算节点之间拆分工作，与多个并行化只会增加查询性能。 如果发现缩放未更改你的性能，请查看我们的性能优化文章，以检查是否没有数据均匀分布，或者如果正在引入大量的数据移动。 

### <a name="when-should-i-scale-dwus"></a>何时应进行 DWU 缩放？
缩放 DWU 会改变以下重要方案：

1. 以线性方式更改系统对扫描、聚合和 CTAS 语句的性能
2. 使用 PolyBase 加载时增加读取器和编写器的数
3. 并发查询和并发槽的最大数量

关于何时进行 DWU 缩放的建议：

1. 执行繁重的数据加载或转换操作时，增加 DWU 数目即可更快速地获取数据。
2. 高峰业务时段，扩大规模以容纳更大数量的并发查询。 

## <a name="pause-compute-bk"></a> 暂停计算
[AZURE.INCLUDE [SQL Data Warehouse pause description](../../includes/sql-data-warehouse-pause-description.md)]

若要暂停数据库，请使用下列任意方法之一。

* [通过 Azure 门户暂停计算][Pause compute with Azure portal]
* [通过 PowerShell 暂停计算][Pause compute with PowerShell]
* [通过 REST API 暂停计算][Pause compute with REST APIs]

## <a name="resume-compute-bk"></a> 恢复计算
[AZURE.INCLUDE [SQL Data Warehouse resume description](../../includes/sql-data-warehouse-resume-description.md)]

若要恢复数据库，请使用下列任意方法之一。

* [通过 Azure 门户恢复计算][Resume compute with Azure portal]
* [通过 PowerShell 恢复计算][Resume compute with PowerShell]
* [通过 REST API 恢复计算][Resume compute with REST APIs]

## <a name="check-compute-bk"></a>检查数据库状态 
若要恢复数据库，请使用下列任意方法之一。

- [使用 T-SQL 检查数据库状态][Check database state with T-SQL]
- [使用 PowerShell 检查数据库状态][Check database state with PowerShell]
- [使用 REST API 检查数据库状态][Check database state with REST APIs]

## <a name="permissions"></a>权限

缩放数据库需要 [ALTER DATABASE][ALTER DATABASE] 中所述的权限。  暂停和恢复需要 [SQL DB 参与者][SQL DB Contributor]权限，具体说来就是 Microsoft.Sql/servers/databases/action。

## <a name="next-steps-bk"></a> 后续步骤
请参阅以下文章，了解其他一些重要性能概念：

* [工作负荷和并发管理][Workload and concurrency management]
* [表设计概述][Table design overview]
* [表分布][Table distribution]
* [表索引][Table indexing]
* [表分区][Table partitioning]
* [表统计信息][Table statistics]
* [最佳实践][Best practices]

<!--Image reference-->

<!--Article references-->
[data warehouse units (DWUs)]: /documentation/articles/sql-data-warehouse-overview-what-is/#predictable-and-scalable-performance-with-data-warehouse-units
[billed]: /pricing/details/sql-data-warehouse/
[linearly]: /documentation/articles/sql-data-warehouse-overview-what-is/#predictable-and-scalable-performance-with-data-warehouse-units
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

[Check database state with T-SQL]: /documentation/articles/sql-data-warehouse-manage-compute-tsql/#check-database-state-and-operation-progress
[Check database state with PowerShell]: /documentation/articles/sql-data-warehouse-manage-compute-powershell/#check-database-state
[Check database state with REST APIs]: /documentation/articles/sql-data-warehouse-manage-compute-rest-api/#check-database-state

[Workload and concurrency management]: /documentation/articles/sql-data-warehouse-develop-concurrency/
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

<!--Update_Description:update meta properties,wording update; update reference; add how to operation work in the sql data warehouse-->