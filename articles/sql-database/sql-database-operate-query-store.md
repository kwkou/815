<properties
   pageTitle="在 Azure SQL 数据库中操作 Query Store"
   description="了解如何在 Azure SQL 数据库中操作 Query Store"
   keywords=""
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jhubbard"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="05/25/2016"
   wacn.date="07/11/2016"/>

# 在 Azure SQL 数据库中操作 Query Store 

Azure 中的 Query Store 是完全托管的数据库功能，可持续收集和提供有关所有查询的详细历史信息。可以将 Query Store 视为一个航班数据记录器，它可以大幅简化云与本地客户的查询性能故障排除。本文说明在 Azure 中操作 Query Store 的具体方法。使用这些预先收集的查询数据，用户可以快速诊断并解决性能问题，因此将更多的时间投入到业务上。

从 2015 年 11 月开始，Query Store 已在 Azure SQL 数据库中[全球推出](https://azure.microsoft.com/updates/general-availability-azure-sql-database-query-store/)。Query Store 是性能分析和优化功能的基础。在本文发布时，Query Store 正在 Azure 中运行 200,000 多个用户数据库，不间断地收集多个月的查询相关信息。

> [AZURE.IMPORTANT] Microsoft 即将为所有 Azure SQL 数据库（现有的和新的）激活 Query Store。此激活过程目前只包括单一数据库，但不久之后，将扩展到弹性池数据库。使用弹性池时，只能为一小部分数据库启用 Query Store。为弹性池中的所有数据库启用 Query Store 可能会导致资源用量过高，并可能导致系统无响应。

## Query Store 是 Azure SQL 数据库中的托管功能

Query Store 在 Azure SQL 数据库中基于以下两个基本原则运行：

- 对客户工作负荷不造成任何可见的影响
- 除非客户工作负荷受影响，否则持续监视查询

对客户工作负荷的影响体现在两个方面：

- **可用性**：运行 Query Store 时，不降低 [SQL 数据库的 SLA](/support/sla/sql-data)。
- **性能**：Query Store 产生的平均开销通常在 1-2% 的范围。

Azure 中的 Query Store 在运行时使用有限的资源（CPU、内存、磁盘 I/O、磁盘大小等）。为了最大程度地降低将对一般工作负荷的影响，Query Store 遵守各种系统限制：

- **静态限制**：给定服务层（基本、标准、高级、弹性池）的资源容量施加的限制。
- **动态限制**：当前工作负荷消耗量（即可用资源）施加的限制。

为了确保连续可靠的操作，Azure SQL 数据库围绕 Query Store 构建了永久性监视基础结构，用于从每个数据库收集关键操作数据。因此，会不断监视多个技术 KPI 以确保可靠性：

- 异常和自动缓解的数目
- 处于 READ\_ONLY 状态的数据库数目和 READ\_ONLY 状态的持续时间
- Query Store 内存消耗量超过限制的前几个数据库。
- 根据自动清理频率和持续时间列出的前几个数据库
- 根据将数据载入内存（Query Store 初始化）所花费的持续时间列出的前几个数据库
- 根据将数据刷新到磁盘所花费的持续时间列出的前几个数据库

Azure SQL 数据库使用收集的数据来实现以下目的：

- **了解大量数据库的使用模式，并以此改善功能可靠性和质量：**Query Store 随 Azure SQL 数据库的每次更新得到改善。 
- **解决或缓解 Query Store 所造成的问题：**Azure SQL 数据库可以检测并缓解对客户工作负荷带来实质影响的问题，且不会造成太大的延迟（小于一小时）。暂时将 Query Store 设置为“关闭”往往就能解决问题。

有时，Query Store 更新会造成应用到内部和极少外部（面向客户）配置默认值发生更改。因此，客户在 Azure SQL 数据库中使用 Query Store 的体验可能因 Azure 平台自动执行的操作而与本地环境中的体验有所不同：

- 可以将 Query Store 状态更改为“关闭”以缓解问题，然后在问题解决后，再更改回到“打开”。
- 可以更改 Query Store 配置以确保其可靠工作。执行的更改包括：
    - 在单个数据库方面进行更改，以解决不稳定性或不可靠性问题。
    - 全局实施最佳配置更改，以便为使用 Query Store 的所有数据库提供优势。

将 Query Store 设置为“关闭”是范围局限于单个数据库的自动操作。当产品行为对用户数据库造成负面影响，使得检测机制触发警报时，将执行这种操作。对于该特定数据库，Query Store 将一直保持“关闭”，直到有改进了 Query Store 实现的新版本可用为止。在转换为“关闭”状态后，将通过电子邮件向客户发送通知，并建议客户在有新版本推出之前避免重新启用 Query Store。推出新版本后，Azure SQL 数据库基础结构将针对其 Query Store 设置为“关闭”的所有数据库自动激活 Query Store。

有时（不太常见），Azure SQL 数据库可能会对所有用户数据库强制实施新的配置默认值，这些默认值经过优化，以确保运行可靠并能够持续收集数据。

## 最佳的 Query Store 配置

本部分描述最佳的配置默认值，这些默认值旨在确保 Query Store 以及依赖功能能够可靠运行。默认配置已针对持续数据收集操作进行优化，即，在 OFF/READ\_ONLY 状态下花费最少的时间。

| 配置 | 说明 | 默认 | 注释 |
| ------------- | ----------- | ------- | ------- |
| MAX\_STORAGE\_SIZE\_MB | 指定 Query Store 在客户数据库中占用的数据空间的限制 | 100 | 对新数据库强制实施 |
| INTERVAL\_LENGTH\_MINUTES | 定义聚合和持久化查询计划收集运行时统计信息的时段大小。每个活动查询计划将为此配置定义的时间段包含最多一行 | 60 | 对新数据库强制实施 |
| STALE\_QUERY\_THRESHOLD\_DAYS | 基于时间的清理策略，控制持久化运行时统计信息和非活动查询的保留期 | 30 | 对新数据库和使用以前的默认值 (367) 的数据库强制实施 |
| SIZE\_BASED\_CLEANUP\_MODE | 指定当 Query Store 数据大小接近限制时是否自动清理数据 | AUTO | 对所有数据库强制实施 |
| QUERY\_CAPTURE\_MODE | 指定是要跟踪所有查询，还是只跟踪一部分查询 | AUTO | 对所有数据库强制实施 |
| FLUSH\_INTERVAL\_SECONDS | 指定捕获的运行时统计信息在刷新到磁盘之前，保留在内存中的最大期限 | 900 | 对新数据库强制实施 |
||||||

> [AZURE.IMPORTANT] 在 Query Store 的最终激活阶段，将在所有 Azure SQL 数据库中自动应用这些默认值（请参阅上面的重要说明）。激活后，Azure SQL 数据库不会更改客户设置的配置值，除非这些值对主要工作负荷或 Query Store 的可靠运行造成负面影响。

如果想要保持使用自定义设置，请[结合 Query Store 选项使用 ALTER DATABASE](https://msdn.microsoft.com/zh-cn/library/bb522682.aspx)，以将配置还原到以前的状态。请查看 [Best Practices with the Query Store（Query Store 最佳实践）](https://msdn.microsoft.com/zh-cn/library/mt604821.aspx)，以了解如何选择最佳的配置参数。

## 后续步骤

[SQL 数据库性能见解](/documentation/articles/sql-database-performance-guidance)

## 其他资源

有关详细信息，请查看以下文章：

- [A flight data recorder for your database（数据库的航班数据记录器）](https://azure.microsoft.com/blog/query-store-a-flight-data-recorder-for-your-database) 

- [Monitoring Performance By Using the Query Store（使用 Query Store 监视性能）](https://msdn.microsoft.com/zh-cn/library/dn817826.aspx)

- [Query Store Usage Scenarios（Query Store 使用方案）](https://msdn.microsoft.com/zh-cn/library/mt614796.aspx)

- [Monitoring Performance By Using the Query Store（使用 Query Store 监视性能）](https://msdn.microsoft.com/zh-cn/library/dn817826.aspx)

<!---HONumber=Mooncake_0704_2016-->