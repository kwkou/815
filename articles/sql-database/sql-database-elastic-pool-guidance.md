<properties
    pageTitle="何时使用弹性池？"
    description="弹性池是由一组弹性数据库共享的可用资源集合。本文档提供相关指导来帮助你评估是否适合对一组数据库使用弹性池。"
    services="sql-database"
    documentationcenter=""
    author="CarlRabeler"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="3d3941d5-276c-4fd2-9cc1-9fe8b1e4c96c"
    ms.service="sql-database"
    ms.custom="multiple databases"
    ms.devlang="NA"
    ms.date="12/19/2016"
    wacn.date="01/20/2017"
    ms.author="sstein;carlrab"
    ms.workload="data-management"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="NA" />  


# 何时使用弹性池？
根据弹性池和单一数据库之间的数据库使用模式和定价区别，评估使用弹性池是否具有成本效益。此外，还提供了其他指导来帮助确定一组现有 SQL 数据库所需的当前池大小。

- 有关池的概述，请参阅 [SQL 数据库弹性池](/documentation/articles/sql-database-elastic-pool/)。

> [AZURE.NOTE]
弹性池已正式发布 (GA)。
> 
> 

## 弹性池
SaaS 开发人员构建在由多个数据库组成的大规模数据层上的应用程序。常见的应用程序模式是为每位客户设置单一数据库。但不同的客户通常拥有不同和不可预测的使用模式，很难预测每位数据库用户的资源需求。所以开发人员可能以可观的费用过度预配资源，以确保所有的数据库能有最佳吞吐量和响应时间。或者，开发人员可以使用较少的费用，让客户承担性能体验不佳的风险。若要了解有关使用弹性池的 SaaS 应用程序的设计模式的详细信息，请参阅[多租户 SaaS 应用程序和 Azure SQL 数据库的设计模式](/documentation/articles/sql-database-design-patterns-multi-tenancy-saas-applications/)。

Azure SQL 数据库中的弹性池可使 SaaS 开发人员将一组数据库的价格性能优化在规定的预算内，同时为每个数据库提供性能弹性。池可让开发人员为由多个数据库共享的池购买弹性数据库交易单位 (eDTU)，以适应单一数据库使用时段不可预测的情况。池的 eDTU 要求取决于其数据库的聚合使用量。池可用的 eDTU 数量由开发人员预算控制。池可让开发人员轻松为他们的池推断预算对性能的影响，反之亦然。开发人员只需将数据库添加到池，为数据库设置 eDTU 最小值和最大值，然后根据其预算设置池的 eDTU。开发人员可以使用池顺畅地扩大其服务，以渐增的规模从精简的新创公司发展到成熟的企业。

## 何时应考虑使用池
池很适合具有特定使用模式的大量数据库。对于给定的数据库，此模式的特征是低平均使用量与相对不频繁的使用高峰。

可以加入池的数据库越多，就可以节省更多的成本。具体取决于应用程序使用模式，你可能会看到与使用两个 S3 数据库相同的成本节约。

以下各部分有助于了解如何评估特定的数据库集合是否会因使用池而受益。这些示例使用标准池，但同样的原理也适用于基本和高级池。

### 评估数据库使用模式
下图显示了一个数据库示例，该数据库有大量的闲置时间，但也会定期出现活动高峰。这是适合池的使用模式：

   ![适用于池的单一数据库](./media/sql-database-elastic-pool-guidance/one-database.png)  


在所示的五分钟时间段内，DB1 高峰最高达到 90 个 DTU，但其整体平均使用量低于五个 DTU。在单一数据库中，运行此工作负荷需要 S3 性能级别，但在低活动期间，大多数资源都处在未使用的状态。

池可让这些未使用的 DTU 跨多个数据库共享，因此减少了所需的 DTU 数和总体成本。

以上一个示例为基础，假设有其他数据库具有与 DB1 类似的使用模式。在接下来的两个图形中，4 个数据库和 20 个数据库的使用量分层放在相同的图形上，以说明随时间推移，它们的使用率非重叠的性质：

   ![使用模式适用于池的 4 个数据库](./media/sql-database-elastic-pool-guidance/four-databases.png)

   ![使用模式适用于池的 20 个数据库](./media/sql-database-elastic-pool-guidance/twenty-databases.png)  


在上图中，黑线表示跨所有 20 个数据库的聚合 DTU 使用量。其中表明聚合 DTU 使用量永远不会超过 100 个 DTU，并指出 20 个数据库可以在此时间段内共享 100 个 eDTU。相比于将每个数据库放在单一数据库的 S3 性能级别，这会导致 DTU 减少 20 倍，价格降低 13 倍。

由于以下原因，此示例很理想：

* 每一数据库之间的高峰使用量和平均使用量有相当大的差异。
* 每个数据库的高峰使用量在不同时间点发生。
* eDTU 将在多个数据库之间共享。

池的价格取决于池的 eDTU。尽管池的 eDTU 单位价格比单一数据库的 DTU 单位价格多 1.5 倍，但**池 eDTU 可由多个数据库共享，因而所需的 eDTU 总数更少**。定价方面和 eDTU 共享的这些差异是池可以提供成本节省可能性的基础。

以下数据库计数和数据库使用率相关规则的经验法则可帮助确保池提供相比于使用单一数据库的性能级别降低的成本。

### 数据库的最小数目
如果单一数据库性能级别的 DTU 总和比池所需的 eDTU 多 1.5 倍，则弹性池更具成本效益。有关可用的大小，请参阅[弹性池和弹性数据库的 eDTU 和存储限制](/documentation/articles/sql-database-elastic-pool/#edtu-and-storage-limits-for-elastic-pools-and-elastic-databases)。

**示例** <br> 
至少需要 2 个 S3 数据库或 15 个 S0 数据库，才能使 100 个 eDTU 池比使用单一数据库性能级别更具成本效益。

### 并发高峰数据库的最大数目
通过共享 eDTU，并非池中的所有数据库都能同时使用 eDTU 达到使用单一数据库的性能级别时的最大限制。并发高峰的数据库越少，可以设置的池 eDTU 就越低，也就能实现池更大的成本效益。一般而言，池中不能有 2/3（或 67%）以上的数据库的高峰同时达到其 eDTU 限制。

**示例** <br> 
为了降低 200 个 eDTU 池内 3 个 S3 数据库的成本，在使用过程中最多可使其中两个数据库同时处于高峰。否则，如果四个 S3 数据库中超过两个同时高峰，则必须将池缩放为超过 200 个 eDTU。如果将池的大小调整为超过 200 个 eDTU，则需要将更多的 S3 数据库加入到池中，才能使成本保持低于单一数据库的性能级别。

请注意，此示例未考虑池中其他数据库的使用率。如果在任何给定时间点，所有数据库都有一些使用量，则可以同时处于高峰的数据库应少于 2/3（或 67%）。

### 每个数据库的 DTU 使用率
数据库的高峰和平均使用率之间的差异为，长时间的低使用率和短时间的高使用率。这个使用模式非常适合在数据库之间共享资源。当数据库的高峰使用率比平均使用率大 1.5 倍左右时，应考虑将数据库用作池。

**示例**<br>
高峰为 100 个 DTU 且平均使用 67 个或更少 DTU 的 S3 数据库是在池中共享 eDTU 的良好候选项。或者，高峰为 20 个 DTU 且平均使用 13 个或更少 DTU 的 S1 数据库是池的良好候选项。

## 调整弹性池的大小
池的最佳大小取决于聚合 eDTU 和池中所有数据库所需的存储资源。这涉及到决定以下两个数量的较大值：

* 池中所有数据库使用的最大 DTU。
* 池中所有数据库使用的最大存储字节。

有关可用的大小，请参阅[弹性池和弹性数据库的 eDTU 和存储限制](/documentation/articles/sql-database-elastic-pool/#eDTU-and-storage-limits-for-elastic-pools-and-elastic-databases)。

SQL数据库自动评估现有 SQL 数据库服务器中数据库的历史资源使用率，并在 Azure 门户中推荐适当的池配置。除推荐外，内置体验还会估算服务器上自定义组数据库的 eDTU 使用率。这样可以执行“假设”分析，其方法为：通过交互方式将数据库添加到池并删除它们以在提交所做的更改之前获取资源使用率分析和调整建议。相关操作方式，请参阅[监视、管理弹性池并调整其大小](/documentation/articles/sql-database-elastic-pool-manage-portal/)。

若要获得更灵活的资源使用率评估，以便估算 V12 之前版本的服务器的临时大小调整，并估算其他服务器中数据库的大小调整，请参阅[可标识适用于弹性池的数据库的 Powershell 脚本](/documentation/articles/sql-database-elastic-pool-database-assessment-powershell/)。

| 功能 | 门户体验 | PowerShell 脚本 |
|:--- |:--- |:--- |
| 粒度 |15 秒 |15 秒 |
| 考虑池与单一数据库性能级别之间的定价差异 |是 |否 |
| 允许自定义已分析的数据库列表 |是 |是 |
| 允许自定义分析中使用的时间段 |否 |是 |
| 允许自定义跨不同服务器的已分析数据库列表 |否 |是 |
| 允许自定义 v11 服务器上的已分析数据库列表 |否 |是 |

在无法使用工具的情况下，以下分步步骤有助于评估池是否比单一数据库更具成本效益：

1.	通过如下方式来估算池所需的 eDTU：

    MAX（<数据库的总数目 X 每一数据库的平均 DTU 使用率>、<br>
    <并发高峰数据库的数目 X 每一数据库的高峰 DTU 使用率）

2.	通过将池内所有的数据库所需的字节数相加来估算池所需要的存储空间。然后，确定提供此存储量的 eDTU 池的大小。有关基于 eDTU 池大小的池存储限制，请参阅[弹性池和弹性数据库的 eDTU 和存储限制](/documentation/articles/sql-database-elastic-pool/#eDTU-and-storage-limits-for-elastic-pools-and-elastic-databases)。
3.	选择步骤 1 和步骤 2 中 eDTU 估算值中较大的那个。
4.	请参阅 [SQL 数据库定价页面](/pricing/details/sql-database/)，查找大于步骤 3 中估计值的最低 eDTU 池大小。
5.	将步骤 5 的池价格与单一数据库适当性能级别的价格相比较。

## 摘要
并非所有单一数据库都是池的最佳候选项。使用模式表现为低平均使用率和相对不出现频繁使用高峰的数据库，是池的绝佳候选项。应用程序使用模式是动态的，因此，请使用本文中所述的信息和工具来进行初始评估，以确定池是否是部分或所有数据库的良好选择。本文仅是帮助你确定弹性池是否合适的起点。请记住，应持续监视历史资源使用量，并不断地重新评估所有数据库的性能级别。需要记住的是，你可以轻松地将数据库移入和移出弹性池。如果拥有大量数据库，则可以创建大小不同的多个池，以将数据库划分到其中。

## 后续步骤
- [创建弹性池](/documentation/articles/sql-database-elastic-pool-create-portal/)
- [监视、管理弹性池并调整其大小](/documentation/articles/sql-database-elastic-pool-manage-portal/)
- [SQL 数据库选项和性能：了解每个服务层中可用的功能](/documentation/articles/sql-database-service-tiers/)
- [可标识适用于弹性池的数据库的 Powershell 脚本](/documentation/articles/sql-database-elastic-pool-database-assessment-powershell/)

<!---HONumber=Mooncake_0116_2017-->
<!--update: tanslation update "弹性数据库池" to "弹性池"-->